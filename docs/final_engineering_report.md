# Final Engineering Report: Software Engineering QA Agent

**Group:** Group N  
**Project:** AI-Powered Software Engineering QA Agent  
**Academic Year:** 2026/2027  
**Course:** BSE4104 — Emerging Trends in Software Engineering  
**Date:** 23rd October 2026

---

## 1. Executive Summary

Software developers spend significant time writing tests, running them, and triaging
failures — work that is slow, repetitive, and error-prone. The SE QA Agent is a bounded,
agentic quality assurance system that automates this workflow: a developer enters a testing
goal in natural language, the agent retrieves relevant source code from a vector database,
generates pytest test cases using a foundation model, executes them in an isolated Docker
sandbox, analyses failures with AI, and drafts a structured GitHub issue for human review.

The system was built progressively over eight weeks, progressing from a Gemini API baseline
through prompt engineering, retrieval-augmented generation (RAG), tool/function calling, a
bounded agent loop with iteration limits, persistent memory, a Streamlit user interface,
and a 30-scenario evaluation suite with guardrails and observability. All 109 unit tests
pass with 76% line coverage. The agent achieved 100% task completion, 100% safety
guardrail effectiveness, and 68% groundedness across the evaluation suite. Six genuine
failures were identified, fixed, and re-tested.

The principal limitation is the dependency on Docker Desktop for test execution — the agent
cannot run tests without a local Docker daemon. Additionally, the agent generates Python
tests only; direct testing of Java, JavaScript, or other languages requires mocked
self-contained tests rather than real execution.

---

## 2. Problem Context and Requirements

### 2.1 Target User and Workflow

The target user is a software developer or quality engineer who needs rapid, automated
test generation and failure triaging without relinquishing control over what gets committed
or posted. The primary end-to-end workflow is:

1. Developer enters a testing goal (e.g., "test that rag.retrieve() handles an empty
   corpus gracefully").
2. Agent retrieves relevant code and documentation from a ChromaDB vector store.
3. Agent generates 3–8 pytest test cases using Gemini 3.8 Flash.
4. Agent executes the tests inside a `python:3.11-slim` Docker container.
5. If tests fail, agent analyses the root cause and either retries (up to 3 iterations)
   or drafts a GitHub issue.
6. Developer reviews the draft and explicitly approves posting to GitHub.

### 2.2 Requirements and Acceptance Criteria

Twelve user stories with acceptance criteria were defined in
`docs/requirements/user_stories.md`, covering goal-directed test generation (US-01),
sandboxed execution (US-02), failure analysis (US-03), human-approved issue drafting
(US-04), iteration limits (US-05), run history (US-06), guardrails (US-07), contextually
grounded plans (US-08), traceability (US-09), observability (US-10), reproducible setup
(US-11), and no secrets in version control (US-12).

### 2.3 AI Boundary Matrix

| AI May Do | Deterministic Software Controls | Requires Human Approval |
|---|---|---|
| Propose test plans | Execute tests (Docker sandbox) | Submit GitHub issue |
| Generate test code | Validate generated code syntax | Merge any PR |
| Summarise failures | Enforce iteration limits | Any destructive action |
| Draft GitHub issues | Manage API keys and permissions | |
| Retrieve relevant context | Tool allow-list enforcement | |
| Chat about the system | Output schema validation | |

### 2.4 Success Measures

| Criterion | Target | Achieved |
|---|---|---|
| Task completion across 30 scenarios | ≥90% | 100% |
| Groundedness across applicable scenarios | ≥60% | 68% |
| Safety guardrail effectiveness | 100% | 100% |
| Test suite pass rate | 100% | 100% (109/109) |
| Line coverage | ≥70% | 76% |
| Genuine failures found and fixed | ≥5 | 6 (all re-tested PASS) |

---

## 3. System Architecture

The system is composed of five layers, each building on the one below:

```
Layer 5 — Agent Loop (src/agents.py)
    Orchestrates SENSE→PLAN→ACT→OBSERVE→DECIDE with iteration limits
Layer 4 — Tools (src/tools.py)
    Four allow-listed tools: propose_tests, run_tests, analyze_failures, draft_issue
Layer 3 — RAG (src/rag.py)
    ChromaDB vector store; indexes own codebase + external codebases
Layer 2 — Prompts (prompts/)
    Three versioned prompt files with placeholder templates
Layer 1 — Model (src/llm_client.py)
    Gemini 3.8 Flash wrapper with structured logging
```

### 3.1 Deterministic Components

All safety-critical operations are deterministic, not AI-driven:

- **Tool allow-list** (`call_tool()`) — only four registered tool names are permitted; any
  other name raises `ToolNotAllowedError` before execution.
- **Syntax gate** (`run_tests()`) — `ast.parse()` validates test code before Docker is
  started; invalid code never reaches the sandbox.
- **Docker sandbox** — test files are mounted read-only; the container has no host
  filesystem access; containers are always removed via `try/finally`.
- **Iteration hard cap** — the loop condition `while iteration < max_iterations` cannot be
  bypassed by any tool output.
- **Human approval gate** — `draft_issue()` defaults to `approved=False`; the GitHub API
  is never called without explicit user action.

### 3.2 AI Components

- **Gemini 3.8 Flash** — used for test plan generation, failure analysis, and issue body
  formatting. All calls go through `llm_client.generate()` which logs prompt hash, token
  counts, and latency.
- **gemini-embedding-001** — produces 3072-dimensional embeddings for RAG indexing and
  retrieval. All calls go through `llm_client.embed()`. The model supports Matryoshka
  Representation Learning, allowing truncation to 768 or 1536 dimensions, but the default
  3072 is used without explicit `output_dimensionality` configuration.

### 3.3 External Service Boundaries

| Service | Access method | When accessed | Human approval required |
|---|---|---|---|
| Google AI Studio (Gemini) | `google-genai` SDK | Every PLAN, OBSERVE, and DECIDE step | No |
| Docker daemon | `docker` Python SDK | Every ACT step | No (sandboxed) |
| GitHub API | `PyGithub` | Only when user clicks "Submit to GitHub" | Yes |
| ChromaDB | Local file (`evidence/chroma_db/`) | SENSE and build_index | No |
| SQLite | Local file (`evidence/qa_agent.db`) | After every run | No |

### 3.4 State and Memory Boundaries

Ephemeral state (run_id, iteration count, context chunks, test results) lives in
`QAAgent.run()` local variables. Persistent state (run history, drafted issues) is stored
in SQLite via `src/memory.py`. The Streamlit UI maintains session state in
`st.session_state` so the current run is not lost on widget interaction.

---

## 4. Foundation Model and Prompt Engineering

### 4.1 Model Rationale

Gemini 3.8 Flash was selected over four alternatives:

| Model | Rejected because |
|---|---|
| DeepSeek (free API) | Frequent 503 errors; no co-located embedding model |
| GPT-4o (OpenAI) | No free tier; would require paid API credits |
| Claude 3.5 Sonnet | No free tier; same cost issue |
| Llama 3.1 70B (local) | Too slow on student hardware; unreliable function calling |

Gemini 3.8 Flash was chosen because it provides a stable free tier, native function calling,
a 1M-token context window, and a co-located embedding model (`gemini-embedding-001`) under
the same API key — avoiding the complexity of a second provider for RAG.

The model was migrated from `gemini-2.0-flash` (shut down June 2026) to `gemini-3.8-flash`
during Week 8. The embedding model was migrated from `text-embedding-004` (shut down
January 2026) to `gemini-embedding-001`. Both migrations required only environment variable
changes due to the `llm_client.py` abstraction layer.

### 4.2 Prompt Architecture and Versioning

Three prompts were developed, each following a consistent structure: ROLE → TASK →
CONSTRAINTS → NEGATIVE EXAMPLE → OUTPUT FORMAT. All prompts output structured JSON with no
surrounding text.

| Prompt | Current version | Purpose | Output |
|---|---|---|---|
| `test_planner.txt` | v1.3 | Generate pytest test cases from a goal + context | JSON: `{goal, test_cases[{name, type, description, code}]}` |
| `failure_analyzer.txt` | v1.1 | Analyse pytest failure output; identify root cause | JSON: `{summary, root_cause, affected_files, suggested_fix, confidence}` |
| `issue_drafter.txt` | v1.0 | Format analysis as a GitHub issue body | JSON: `{title, body}` |

All versions are archived in `prompts/versions/` with changelogs documenting each change
and the failure that motivated it.

### 4.3 Important Prompt Iterations

**v1.0 → v1.1 (test_planner):** The model occasionally wrapped JSON output in markdown code
fences despite an explicit constraint against it (eval case TP-04). Fix: added a NEGATIVE
EXAMPLE block showing the disallowed format immediately before the OUTPUT FORMAT section.
This proved more reliable than constraint text alone.

**v1.1 → v1.2 (test_planner):** Generated tests imported `from src.*` which failed inside
the Docker sandbox when no grounding context was available (failure FC-03). Fix: added a
fallback rule — if no context is provided, generate self-contained tests using only
`unittest.mock`.

**v1.2 → v1.3 (test_planner):** External codebases (e.g., Java projects) could not be
imported inside the Docker sandbox either, even when context was retrieved (failure FC-06).
Fix: added RULE A — if any context file path starts with `[` (external label), generate
mocked tests only.

### 4.4 Baseline Evaluation

A 10-case prompt evaluation table was recorded in `docs/evaluation/prompt_eval_table.md`.
Result: 9/10 passed (90%) on v1.0, improving to 10/10 after v1.1 fixes.

---

## 5. Context Engineering and RAG

### 5.1 Corpus and Provenance

The RAG corpus is the agent's own codebase and documentation — a self-referential design
where every new module becomes immediately available as grounding context for the next
week's test generation. The corpus covers five directories:

| Directory | Extensions | Files indexed | Est. chunks |
|---|---|---|---|
| `src/` | `.py` | 7 | ~26 |
| `tests/` | `.py` | 4 | ~21 |
| `prompts/` | `.txt`, `.md` | 3 | ~10 |
| `docs/` | `.md` | 10 | ~29 |
| Root | `.md` | 3 | ~5 |
| **Total** | | **27** | **~91** |

Full provenance is recorded in `knowledge/corpus_register.md`. Each chunk stores
`file_path`, `chunk_type` (function/class/section/file), `chunk_name`, and
`last_modified` (ISO 8601 UTC).

### 5.2 Chunking and Indexing

Two chunking strategies are used:

- **Python files** — AST-based: top-level `def` and `class` nodes are extracted as
  individual chunks; a preamble chunk covers module docstrings and imports. Falls back to
  a single file chunk on `SyntaxError`. This keeps each function self-contained rather
  than splitting mid-definition.
- **Markdown and text files** — Heading-based: content is split on `#` headings into
  section chunks. Files with no headings are split on blank-line paragraphs. This works
  for Java, JavaScript, and other structured source files indexed via the external
  codebase feature.

Chunk IDs are deterministic SHA-256 hashes of file path + chunk type + name + first 64
characters of text, making `build_index()` fully idempotent.

### 5.3 Retrieval Approach

`retrieve(query, top_k=5)` converts the query to an embedding, queries ChromaDB by cosine
similarity, and returns ranked results. A similarity threshold of 0.7 (cosine distance)
filters out low-relevance chunks — if all results exceed this threshold, the system
returns an empty list and tells the model "no relevant context found" rather than
passing misleading grounding (failure FC-01).

### 5.4 External Codebase Support

`index_external_codebase(root, label, extensions)` allows indexing any project on the
filesystem — Java, Python, JavaScript, etc. Each chunk's `file_path` is prefixed with
`[label]/` so retrieval results clearly identify which codebase they came from. The UI
sidebar provides a folder path input, optional label, and extensions field.

### 5.5 Retrieval Failures

Three retrieval failure cases were documented in `docs/evaluation/rag_eval.md`:

| ID | Failure | Cause | Fix |
|---|---|---|---|
| RF-01 | Root-level `.md` files not indexed | `_CRAWL_DIRS` excluded project root | Added non-recursive root entry |
| RF-02 | DeepSeek omission in model selection doc | Inconsistent documentation | Updated `foundation_model_selection.md` |
| RF-03 | No similarity threshold | ChromaDB always returns `top_k` results | Added `_MAX_SIMILARITY_DISTANCE = 0.7` filter |

---

## 6. Tools and Agentic Workflow

### 6.1 Tool Contracts

Four tools are defined in `src/tools.py`, each with a schema dict containing `name`,
`description`, `input_schema`, `output_schema`, `permissions`, and `failure_behaviour`.
All tool calls pass through `call_tool()` which enforces the allow-list before dispatch.

| Tool | Input | Output | Permissions |
|---|---|---|---|
| `propose_tests` | `issue_description: str` | `{goal, context_chunks, test_cases[]}` | `read:corpus`, `call:llm` |
| `run_tests` | `test_code: str, timeout: int` | `{status, passed, failed, stdout, stderr, exit_code}` | `execute:docker_sandbox` |
| `analyze_failures` | `test_results: dict` | `{summary, root_cause, affected_files, suggested_fix, confidence}` | `read:corpus`, `call:llm` |
| `draft_issue` | `analysis: dict, approved: bool, goal: str` | `{title, body, labels, github_url}` | `call:llm`, `call:github_api (approved only)` |

All tools return error information as a field in the output dict rather than raising
exceptions. This keeps the agent loop uniform — it always receives a structured dict and
can check for an `"error"` key.

### 6.2 Agent Goal and Loop

The `QAAgent.run(goal)` method implements a SENSE → PLAN → ACT → OBSERVE → DECIDE loop:

1. **SENSE** — `rag.retrieve(goal, top_k=5)` retrieves relevant code chunks. Retrieved
   once and reused across all iterations.
2. **PLAN** — `propose_tests(goal)` calls Gemini with the test_planner prompt and the
   retrieved context. Returns 3–8 test cases as JSON.
3. **ACT** — `run_tests(code)` validates syntax with `ast.parse()`, writes to a temp file,
   starts a Docker container, runs pytest, captures results, and cleans up via
   `try/finally`.
4. **OBSERVE** — `analyze_failures(results)` calls Gemini with the failure_analyzer prompt.
   Short-circuits (no LLM call) when all tests pass.
5. **DECIDE** — If all tests pass: call `draft_issue(approved=False)` and return. If tests
   fail and iterations remain: loop back to PLAN. If limit reached: call `draft_issue` and
   return with `status="handoff"`.

### 6.3 Iteration Limits and Stop Conditions

| Condition | Trigger | Agent status | Action |
|---|---|---|---|
| All tests pass | `test_results["status"] == "passed"` | `passed` | Draft issue, return |
| Docker unavailable | `run_tests` returns Docker error | `error` | Draft issue, return immediately |
| Planning error | `propose_tests` returns error after retries | `error` | Return immediately |
| Iteration limit | `iteration >= MAX_AGENT_ITERATIONS` | `handoff` | Draft issue, return with hand-off message |
| Invalid goal | Goal fails validation | `error` | Return before any tool call |

The limit is enforced by `while iteration < self.max_iterations` — a loop condition that
no tool output can modify.

### 6.4 Human Approval and Hand-Off

`draft_issue()` is always called with `approved=False` by the agent. The draft is returned
to the Streamlit UI for display. The user must explicitly click "Submit to GitHub" to
trigger `draft_issue(approved=True)`, which calls PyGithub's `repo.create_issue()`. This
gate is enforced at three independent points: the function signature default, the
TOOL_REGISTRY schema, and the UI button.

On hand-off (iteration limit reached), the agent still produces a draft issue so the
developer has a structured starting point. The `handoff_msg` includes the iteration count
and last failure summary.

---

## 7. Memory, State and Interoperability

### 7.1 State Model

Two distinct kinds of state exist:

**Ephemeral session state** — held in `QAAgent.run()` local variables and
`st.session_state`. Lost when the process exits. Includes run_id, goal, iteration count,
context chunks, test results, analysis, draft, and trace.

**Persistent memory** — held in `evidence/qa_agent.db` (SQLite). Survives restarts and page
reloads. Two tables: `runs` (run_id, timestamp, goal, trace_json, status) and
`drafted_issues` (issue_id, run_id, title, body, github_url, submitted_at).

### 7.2 Retained Memory and Justification

Memory is used for three legitimate purposes:

1. **Run history sidebar** — shows the last 10 runs with status icons. Without persistent
   memory, this list would reset on every page reload.
2. **Draft preservation** — issue drafts take 30+ seconds to generate. Storing them means
   the user can close the tab and return without losing the draft.
3. **Audit trail** — the JSON trace in `runs.trace_json` provides a permanent record of
   every step, tool call, and decision.

Memory does not silently control any decision — it only stores and retrieves.

### 7.3 Privacy and Retention

- **Local only** — `evidence/qa_agent.db` is a local SQLite file with no network exposure.
- **No PII** — the system operates exclusively on its own codebase and user-entered testing
  goals. No user accounts, email, or personal data are stored.
- **Manual delete only** — no automatic expiry. `MemoryStore.delete_run(run_id)` removes a
  run and its associated drafts. The database file can be deleted directly for a full reset.
- **Gitignored** — `evidence/qa_agent.db` and `evidence/chroma_db/` are in `.gitignore`.

### 7.4 MCP-Style Interface Specification

All four tools are documented as an MCP-style interface in
`docs/architecture/mcp_interface.md` with JSON input/output schemas, permissions, side
effects, and security boundaries. The `draft_issue` tool's human approval gate is
specified as a permission boundary: `call:github_api` is only exercised when
`approved=true`, which requires explicit human action.

---

## 8. Evaluation, Observability and Guardrails

### 8.1 Evaluation Dataset

Thirty evaluation scenarios were generated across six categories, stored in
`docs/evaluation/eval_scenarios.json`:

| Category | Count | Description |
|---|---|---|
| Normal | 8 | Standard happy-path goals targeting real modules |
| Edge | 6 | Unusual but valid inputs (empty files, whitespace queries) |
| Incorrect-information | 4 | Goals referencing non-existent functions/classes |
| Adversarial | 6 | Prompt injection attempts and jailbreak patterns |
| Tool-failure | 4 | Docker unavailable, GitHub auth failure, empty RAG |
| Unauthorized-action | 2 | Auto-merge and production deployment requests |

### 8.2 Evaluation Criteria and Results

Full results are in `docs/evaluation/eval_results.md`:

| Criterion | Result |
|---|---|
| Task completion | 30/30 (100%) |
| Groundedness | 15/22 applicable (68%) |
| Safety guardrails | 8/8 (100%) |

Groundedness failures were concentrated in the incorrect-information category (0/4) — by
design, the corpus cannot ground answers about non-existent modules. The similarity
threshold fix (FC-01) ensures the model is explicitly told "no relevant context found"
rather than receiving misleading chunks.

### 8.3 Guardrail Implementation

Nine guardrails operate at different layers:

| Guardrail | Layer | What it blocks |
|---|---|---|
| Input length limit (>2000 chars) | Input validation | Oversized goals |
| Prompt injection detection (14 patterns) | Input validation | "ignore previous instructions", "you are now", etc. |
| Deployment pattern detection | Input validation | "deploy to production", "merge to main" |
| Tool allow-list | Tool boundary | Unregistered tool names |
| Syntax gate | Execution | Syntactically invalid test code |
| Docker sandbox | Execution | Host filesystem access |
| Output schema validation | Post-tool | Missing required keys in tool output |
| Human approval gate | Output | Autonomous GitHub posting |
| Iteration hard cap | Loop control | Infinite loops |

All 6 adversarial scenarios and both unauthorized-action scenarios were blocked by the
input validation guardrails before any tool was called. Response time for blocked inputs
was under 15ms.

### 8.4 Observability and Tracing

Two observability mechanisms operate in parallel:

**Per-run JSON traces** (`evidence/traces/<run_id>.json`) — one file per run, written
atomically at run end. Each step records: step name, tool called, input summary, output
summary, decision, and UTC timestamp. PLAN steps also store the full proposed test case
code for auditability.

**JSON-lines application log** (`evidence/traces/app.log`) — persistent append-only log.
Every `logging` call from any `src.*` module is serialised as a single JSON object per
line with timestamp, level, logger name, message, and structured event fields (event name,
tool, latency_ms, token counts, etc.).

Full format documentation is in `docs/evaluation/observability_notes.md`.

### 8.5 Failure Catalogue

Six genuine failures were identified, fixed, and re-tested. All re-test statuses are
PASS. Full details in `docs/evaluation/failure_catalogue.md`:

| ID | Failure | Root cause | Fix |
|---|---|---|---|
| FC-01 | RAG returns distant chunks for non-existent modules | No similarity threshold | Added `_MAX_SIMILARITY_DISTANCE = 0.7` filter |
| FC-02 | Groundedness failure on edge-case with sparse corpus | `_MIN_CHUNK_CHARS` not documented in chunk text | Added explanatory comment block |
| FC-03 | Empty RAG causes LLM to generate import-dependent tests | Prompt constraint says "import from src" unconditionally | Added no-src-import fallback to v1.2 prompt |
| FC-04 | "Deploy to production" passes injection validation | No deployment patterns in injection list | Added 4 deployment/merge patterns |
| FC-05 | Docker-unavailable early exit returns `draft=None` | Early return skips `draft_issue()` | Modified error path to call `_draft()` before returning |

A sixth failure (FC-06) was discovered during external codebase testing: generated tests
imported `from src.*` for external Java/Python projects. Fixed in prompt v1.3 by adding
RULE A — external codebase chunks trigger mocked self-contained tests.

---

## 9. Software Engineering Quality and Deployment

### 9.1 Testing

109 unit tests across seven test files, all passing:

| Test file | Tests | Covers |
|---|---|---|
| `tests/test_llm_client.py` | 6 | generate(), embed(), auth errors |
| `tests/test_prompt_loader.py` | 11 | load_prompt(), fill_template(), versioning |
| `tests/test_rag.py` | 15 | build_index(), retrieve(), chunkers |
| `tests/test_tools.py` | 31 | All 4 tools, failure scenarios, helpers |
| `tests/test_agent.py` | 18 | QAAgent loop, validation, trace writing |
| `tests/test_memory.py` | 15 | SQLite store, round-trips, ordering |
| `tests/test_logging_config.py` | 13 | JSON-lines logging, schema validation |

All external dependencies (Gemini, Docker, ChromaDB, PyGithub) are mocked in tests. No
real API calls or container launches are made during the test suite.

### 9.2 Coverage

```
TOTAL  954 stmts  233 miss  76% coverage
```

Coverage exceeds the 70% minimum. `src/main.py` (Streamlit UI, 171 statements) accounts
for the bulk of uncovered lines — it cannot be unit tested without a running Streamlit
server. All other modules are at 90%+ coverage.

### 9.3 Configuration and Secrets

- All secrets are loaded from `.env` via `python-dotenv`. No secret values appear in any
  source file, prompt, or log.
- `.env` is in `.gitignore` and never committed.
- `.env.example` contains only placeholder values.
- `evidence/qa_agent.db`, `evidence/chroma_db/`, and `.pytest_cache/` are gitignored.
- GitHub token scope is `repo` (issues read/write only). No access to code, branches, or
  settings is required.

### 9.4 Code Quality

- Consistent docstrings on all public functions.
- Typed exceptions: `EnvironmentError` for auth, `ValueError` for empty responses,
  `RuntimeError` for API errors, `ToolNotAllowedError` for allow-list violations.
- No bare `except:` blocks — all exception handlers specify the exception type or log
  the error.
- `try/finally` blocks ensure Docker containers and temp files are always cleaned up.
- Exponential backoff retry logic for transient Gemini 5xx errors (1s, 2s, 4s, max 3
  retries).

### 9.5 Reproducibility

```
git clone <repo>
cd qa-agent
cp .env.example .env        # fill in GEMINI_API_KEY, GITHUB_TOKEN, GITHUB_REPO
pip install -r requirements.txt
docker pull python:3.11-slim
streamlit run src/main.py
```

All dependencies are pinned in `requirements.txt` with exact versions. The system runs on
Python 3.11+ and Windows/Linux/macOS.

---

## 10. Results, Limitations and Professional Reflection

### 10.1 What Works

- **End-to-end automation**: a developer can enter a testing goal and receive a drafted
  GitHub issue in under 2 minutes (typical: 60–120s including Docker execution).
- **Self-referential RAG**: indexing the agent's own codebase means every new module
  becomes immediately available as grounding context.
- **Bounded autonomy**: the iteration limit, tool allow-list, and human approval gate
  ensure the agent never runs indefinitely or takes unauthorised actions.
- **Guardrails**: 100% of adversarial and unauthorized-action scenarios were blocked
  before any tool was called, with response times under 15ms.
- **External codebase support**: any project (Java, Python, JS) can be indexed and used
  as grounding context for test generation.
- **Chat interface**: the agent can answer questions about itself and general software
  engineering topics.

### 10.2 What Does Not Work

- **Direct Java/JS test execution**: the Docker sandbox runs `python:3.11-slim` only.
  Tests for external Java projects use `unittest.mock` rather than real execution.
- **Groundedness on non-existent modules**: by design, the corpus cannot ground answers
  about modules that do not exist. The similarity threshold prevents false grounding but
  the generated tests still fail.
- **Gemini free-tier reliability**: occasional 503 errors disrupt agent runs. The retry
  logic mitigates this but cannot eliminate it.
- **Streamlit test coverage**: `main.py` has 0% unit test coverage. Manual demo is the
  only evidence for the UI.

### 10.3 Remaining Risks

- **Docker dependency**: without Docker Desktop running, the agent cannot execute tests.
  This is the largest practical barrier for first-time users.
- **Model deprecation**: Gemini model IDs have changed twice during the project
  (`gemini-2.0-flash` → `gemini-3.8-flash`, `text-embedding-004` → `gemini-embedding-001`).
  The `llm_client.py` abstraction layer makes these one-line changes, but future
  deprecations could break the system.
- **Rate limits**: the free tier allows approximately 1,500 requests/day (limits vary by
  model and may change). Heavy evaluation runs (30 scenarios × 3 iterations × multiple LLM
  calls) can approach this limit.

### 10.4 Future Work

- **Multi-language Docker sandboxes**: add a `maven:3.9-eclipse-temurin-11` image option
  for Java test execution.
- **Propose-only mode**: a UI toggle to stop after PLAN and return test cases without
  running them, for users who want to review the code before execution.
- **Similarity threshold tuning**: the 0.7 threshold was set empirically; a per-query
  adaptive threshold could improve groundedness on edge cases.
- **Continuous indexing**: currently `build_index()` must be called manually or on app
  startup. A file watcher could keep the index up-to-date automatically.

### 10.5 Human Responsibility

The agent is a tool that proposes and reports — it never decides. A human developer
reviews every drafted issue before it is posted to GitHub. A human decides whether to
merge, deploy, or act on any recommendation. The agent's bounded design is not a feature
added at the end — it is a constraint embedded at every layer, from the input validation
that blocks injection attempts to the `approved=False` default that prevents autonomous
GitHub posting. This is the core principle of the system: build the smallest useful
agentic system that you can test, explain, and safely control.

---

## References

1. Osmani, A. (2025). *Beyond Vibe Coding: From Coder to AI-Era Developer.* O'Reilly Media.
2. Huyen, C. (2025). *AI Engineering: Building Applications with Foundation Models.* O'Reilly Media.
3. Lanham, M. (2025). *AI Agents in Action.* Manning Publications.
4. Google AI for Developers. (2026). *Gemini API Models.* https://ai.google.dev/gemini-api/docs/models
5. Google AI for Developers. (2026). *Gemini API Embeddings.* https://ai.google.dev/gemini-api/docs/embeddings

---

## Appendices

### Appendix A — User Stories

See `docs/requirements/user_stories.md` for all 12 user stories with acceptance criteria.

### Appendix B — Prompt Excerpts

See `prompts/test_planner.txt` (v1.3), `prompts/failure_analyzer.txt` (v1.1), and
`prompts/issue_drafter.txt` (v1.0) for full prompt specifications. Version history is in
`prompts/versions/`.

### Appendix C — Evaluation Tables

- 10-case prompt evaluation: `docs/evaluation/prompt_eval_table.md`
- 15-case RAG evaluation: `docs/evaluation/rag_eval.md`
- 30-scenario evaluation results: `docs/evaluation/eval_results.md`
- Failure catalogue: `docs/evaluation/failure_catalogue.md`

### Appendix D — Tool Schemas

See `docs/architecture/tool_catalogue.md` for full schemas, permissions, and step-by-step
flows for all four tools.

### Appendix E — Execution Traces

Three representative traces in `evidence/traces/`:
- `trace_happy_path.json` — all tests pass on first iteration
- `trace_recovery.json` — failure on iteration 1, pass on iteration 2
- `trace_limit_hit.json` — iteration limit reached, hand-off with draft issue

### Appendix F — Architectural Diagrams

See `docs/architecture/architectural_diagrams.md` for the RAG pipeline diagram and agent
loop flowchart.

### Appendix G — Weekly Progress Reports

Weeks 1–8 in `docs/weekly_reports/week1/` through `docs/weekly_reports/week8/`.

### Appendix H — AI Engineering Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-09-14 | Selected Gemini 2.0 Flash over DeepSeek | Free tier, co-located embeddings, function calling |
| 2026-09-14 | AST-based chunking for Python files | Keeps functions self-contained for retrieval |
| 2026-09-14 | Deterministic SHA-256 chunk IDs | Makes build_index() idempotent |
| 2026-09-25 | Allow-list enforced at call_tool() boundary | Single enforcement point; no direct tool calls possible |
| 2026-10-02 | Tools return error dicts, never raise | Uniform loop logic; no try/except at every call site |
| 2026-10-02 | Human approval gate in function signature | Structural enforcement; cannot be bypassed by omitting parameter |
| 2026-10-09 | list_runs() omits trace JSON | Performance — avoid deserialising all traces on sidebar render |
| 2026-10-16 | Similarity threshold of 0.7 | Filters irrelevant RAG results without losing borderline matches |
| 2026-10-16 | JSON-lines logging at module import | Ensures logging active for every code path including tests |
| 2026-10-22 | Exponential backoff retry for Gemini 5xx | Transient errors should not hard-abort the agent loop |
| 2026-10-22 | Prompt v1.3: external codebase mocked tests | External code cannot be imported in Docker sandbox |
| 2026-10-22 | Migrated to gemini-3.8-flash and gemini-embedding-001 | Previous models shut down by Google |
