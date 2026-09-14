
CAPSTONE PROJECT BRIEF
Concise Specification
From AI-Assisted Application to a Bounded, Evaluated Agentic System


Duration	8 weeks
Primary Evidence	GitHub + ClickUp + MUELE
Due Date	Start Date: 31st August 2026- 23rd  October 2026
Presentation Dates	27th, 29th, 30th October 2026
Academic Year	2026/2027

Student learning model — Learn -> Build -> Demonstrate -> Evaluate -> Improve. The objective is not maximum autonomy; it is professional engineering of a useful, bounded and explainable AI-native system.

 
1. Purpose and Learning Outcomes
This capstone converts the AI-native and agentic engineering lecture series into one progressive eight-week project. Teams will engineer a small but credible AI-native application that moves from a foundation-model baseline through context/RAG, tools, bounded agency, state/memory, evaluation and hardening. The project is aligned to BSE4104, which expects students to understand current software innovations and demonstrate practical skill in applying current software solutions to software engineering tasks.
By the end of the project, students should be able to:
1.	Justify where AI adds value and where deterministic software or human decision-making should remain in control.
2.	Integrate and evaluate a foundation model using versioned prompts and explicit acceptance criteria.
3.	Ground model behaviour using trusted context and retrieval rather than relying only on model memory.
4.	Use tools/functions through explicit contracts and permissions.
5.	Implement a bounded agent workflow that can decide among approved actions, observe results and stop safely.
6.	Manage workflow state and justified memory, and understand agent/tool interoperability.
7.	Evaluate reliability, failures, security, observability and guardrails using evidence.
8.	Demonstrate ownership of AI-generated code and artefacts through review, testing, documentation and explanation.
2. Project Scope and Proposal Approval Rules
Each group will propose one problem and progressively extend the same application throughout the eight weeks. The project should be deliberately small enough to finish, test and explain within the semester.
Proposal rule — A chat interface may be part of the product, but the proposal must not be only “a chatbot.” The final system must perform at least one traceable multi-step workflow using approved data/tools and must demonstrate engineering beyond text generation.

A proposal should...	A proposal should NOT...

Solve one clearly bounded problem for a defined user.	Be a generic “assistant for everything” or an entire national/enterprise platform.

Use public, team-created, synthetic or otherwise authorized data that the team can obtain in Week 1.	Depend on confidential institutional databases, proprietary APIs, large datasets or approvals that students do not already have.

Include at least two explicit tools/functions by Week 4.	Simply wrap ChatGPT/Claude/Gemini in a new UI.

Contain one bounded agentic workflow with limits and stop conditions.	Use autonomy merely for novelty, or allow open-ended browsing/execution.

Keep high-impact actions behind human approval.	Make financial, disciplinary, admissions, legal, safety-critical, medical or destructive decisions autonomously.

Be testable using a small controlled corpus and scenario set.	Require training a foundation model from scratch or building an unrealistic amount of infrastructure.

Minimum proposal statement (Week 1): “Our system helps [specific user] complete [specific task]. AI is used for [reasoning/generation/retrieval/planning]. Deterministic software remains responsible for [rules/validation/authorization]. The agent may use [approved tools] but may not [prohibited actions]. We will build and evaluate the system using [available data/corpus].”
3. Recommended Uganda-Relevant Agentic Use Cases
The following are deliberately  selected and scoped for an eight-week project. Teams must select only one of them and may adapt them or propose alternatives of similar complexity. Use public, synthetic or team-created data unless explicit authorization has already been obtained.
Use case	Agentic workflow	Feasible data	Safety boundary
University student-support case agent	Uses approved university/course documents to answer grounded questions, checks a sample timetable or case status, creates/routs a support ticket, and remembers the active case.	Public course/handbook documents + synthetic student/case data.	No admissions, grading, disciplinary or fee decisions.
SME procurement-preparation agent	Reads sample inventory, checks reorder need, compares supplied quotations, drafts a purchase requisition and routes it for human approval.	Synthetic inventory and supplier quotation CSV/JSON.	No real purchasing, payment, supplier award or autonomous financial commitment.
Manufacturing maintenance-triage agent	Grounds troubleshooting in machine manuals, queries mock maintenance history, proposes checks and creates a maintenance ticket.	Equipment manuals + synthetic asset/history records.	No direct machine control, shutdown or safety-critical actuation.
Agricultural extension visit-preparation agent	Uses curated agronomy guidance and a sample farm profile to prepare a field-visit checklist, retrieve relevant advice and schedule/log follow-up.	Public extension/agronomy materials + synthetic farm profiles; optional public weather API.	No pesticide dosing, veterinary/medical diagnosis or autonomous farm-control decisions.
NGO/local-government field-report follow-up agent	Reads sample field reports, classifies issues, extracts actions, assigns a simulated follow-up task and tracks status.	Synthetic/anonymized reports and task records.	No benefit eligibility, enforcement, policing or rights-affecting decisions.
Public procurement document-completeness agent	Checks a sample tender submission against a published checklist, locates required clauses and produces a missing-item report.	Public procurement templates/checklists + synthetic submissions.	No bid scoring, ranking, legal conclusion or contract-award recommendation.
Utility/service help-desk triage agent	Uses a knowledge base to troubleshoot service requests, queries a mock outage/status tool and creates/routes a ticket.	Public help content + synthetic service incidents.	No remote control of infrastructure or automatic service disconnection/reconnection.
SACCO member-case preparation agent	Uses policy documents and synthetic member records to explain procedures, compute illustrative schedules and prepare a case for staff review.	Public/team-created policy + synthetic member data.	No credit scoring, loan approval, disbursement, account changes or real financial transactions.
Software-engineering QA agent	Reads requirements/repository documentation, proposes tests, runs approved tests in a sandbox, summarizes failures and drafts an issue or pull-request note.	Team-owned codebase, requirements and test logs.	No automatic merge, production deployment, secret access or arbitrary shell commands without approval.

Recommended first-time scope — One main user workflow, one small document corpus (about 10-50 documents or equivalent records), two to four tools, one bounded agent loop, one persistent-memory use case, and a 30-scenario final evaluation are sufficient.

4. Minimum Engineering Scope by Week 8
Capability	Minimum evidence
Foundation model	One model integrated through the application; rationale documented.
Prompt engineering	Important prompts stored/versioned; prompt specification and tests.
Context/RAG	Controlled corpus, retrieval, source grounding and unsupported-answer handling.
Tools/function calling	At least two explicit tools; one may retrieve current application data or create a low-risk simulated side effect.
Bounded agent	One goal-directed multi-step loop with approved tools, iteration/stop limits and human hand-off.
State and memory	Workflow/session state plus one justified persistent-memory mechanism.
Interoperability	One external integration or a clear MCP-style tool/interface design.
Evaluation	At least 30 final scenarios including normal, edge, failure and adversarial cases.
Observability/guardrails	Useful logs/traces, validation, permissions, tool allow-list, limits and approval controls.
Software engineering	Git history, tasks/issues, tests, documentation, secure configuration and reproducible run/deployment.

5. Team Organisation, Task Management and Repository
Groups should contain 4-5 students. Roles may rotate. Every member must own identifiable tasks per each week and must understand the whole system well enough to explain major decisions and failures.
1.	Project/Requirements Lead
2.	Application/Integration Lead
3.	AI Engineering Lead
4.	Quality/Security Lead
5.	DevOps/Documentation Lead (for five-person groups)
Task management: You will use ClickUp throughout the project. Create weekly tasks, assign owners, record deadlines and link important tasks to repository evidence.
Recommended GitHub structure
README.md
docs/
  requirements/
  architecture/
  weekly-reports/
  evaluation/
prompts/
knowledge/              # metadata/provenance; do not commit restricted data
src/
tests/
evidence/
  traces/
  screenshots/
  demo/
.env.example             # never commit real secrets
6. AI Use and Professional Integrity
Core rule — AI-generated work is permitted and expected. Unexplained work is not. Students remain accountable for correctness, security, requirements, architecture, testing, documentation and final system behaviour.

1.	Declare the AI tools/models used for design, coding, testing, documentation and evaluation.
2.	Keep a concise AI Engineering Log for material AI-assisted decisions or generated artefacts.
3.	Review, test and refactor generated code before accepting it into the main branch.
4.	Version important prompts and retain evidence of meaningful changes.
5.	Do not send confidential, personal or restricted data to external AI services without authorization.
6.	Do not permit destructive or high-impact actions without explicit human approval controls.
7.	Verify all AI-suggested references and technical claims before using them.
8.	Every student in each group must be able to explain the work they submit.
7. Eight-Week Project Timeline
WEEK 1 (31st August-4th Sept 2026): Problem Framing and AI-Native Requirements
Weekly focus: Choose a feasible problem from the ones proposed in section (3. Recommended Uganda-Relevant Agentic Use Cases) , define users and success criteria, justify AI use, and set the boundaries of the agent.
Activities
•	Select one approved/feasible use case and define a single primary end-to-end workflow.
•	Write a 2-3 page Project Charter: problem, user, current pain point, AI value, scope, assumptions and constraints.
•	Create 8-12 testable user stories/acceptance criteria.
•	Create an AI Boundary Matrix showing what AI may do, what stays deterministic, and what requires human approval.
•	Create the GitHub repository and ClickUp project; assign Week 1 tasks.
Deliverables
•	Project Charter (2-3 pages).
•	User stories and acceptance criteria.
•	AI Boundary Matrix.
•	Initial architecture/context diagram.
•	GitHub + ClickUp setup evidence.
•	Week 1 progress report (1-2 pages).
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 1: “The AI Coding Spectrum,” pp. 4-12; “Programming with Intent,” pp. 13-16; “Benefits and Limitations,” pp. 30-38.
•	Huyen, AI Engineering: Ch. 1: “Foundation Model Use Cases,” pp. 16-28; “Planning AI Applications,” pp. 28-35; “The AI Engineering Stack,” pp. 35-47.
•	Lanham, AI Agents in Action: Ch. 1: “Defining agents,” pp. 1-4; “Component systems of an agent,” pp. 4-9; “Why agents?”, pp. 9-11.
WEEK 2 (7th Sept-11th Sept 2026): Foundation-Model Engineering and Prompting
Weekly focus: Build the smallest useful model-backed capability and establish a tested baseline before adding RAG or agents.
Activities
•	Choose an accessible model and document its capability, cost, latency, privacy and access considerations.
•	Integrate the model into your application.
•	Create Prompt Specification v1.0: role, task, context, constraints, output format and failure behaviour.
•	Create at least 10 test cases and record expected vs actual behaviour.
•	Version at least two meaningful prompt iterations.
Deliverables
•	Working baseline model interaction.
•	Model Selection Note (1 page maximum).
•	Prompt Specification + prompt version history.
•	10-case prompt evaluation table.
•	Week 2 progress report (1-2 pages).
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 1: “AI Models: The Landscape for Code Generation,” pp. 25-30; Ch. 2: prompt engineering, pp. 40-63.
•	Huyen, AI Engineering: Ch. 2: “The Probabilistic Nature of AI,” pp. 105-111; Ch. 5: prompting basics and best practices, pp. 212-235.
•	Lanham, AI Agents in Action: Ch. 2: “Prompting LLMs with prompt engineering,” pp. 25-34; “Choosing the optimal LLM,” pp. 34-36.
WEEK 3 (14th Sept-18th Sept 2026): Context Engineering and RAG
Weekly focus: Make the application answer from a controlled, traceable knowledge source instead of relying only on model memory.
Activities
•	Assemble a controlled corpus (recommended 10-50 documents or equivalent records) and record provenance.
•	Implement ingestion, chunking/segmentation, indexing and retrieval.
•	Construct model context from retrieved evidence and show sources in the response or trace.
•	Create at least 15 RAG test questions: answerable, partially answerable and deliberately unanswerable.
•	Document at least three retrieval/grounding failures and what caused them.
Deliverables
•	Corpus/Source Register.
•	RAG architecture diagram.
•	Working RAG pipeline with source grounding.
•	15-case RAG evaluation results.
•	Week 3 progress report (1-2 pages).
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 2: “Prompting Techniques” and “Advanced Prompting,” pp. 49-63 (supplementary context/decomposition reading; this book does not provide a dedicated RAG chapter).
•	Huyen, AI Engineering: Ch. 6: “RAG,” pp. 253-275, especially RAG architecture, retrieval algorithms and retrieval optimization.
•	Lanham, AI Agents in Action: Ch. 8: “Understanding retrieval in AI applications,” “RAG,” semantic search/document indexing and RAG construction, pp. 181-196.
WEEK 4 (21st Sept-25th Sept 2026): Tools and Function Calling
Weekly focus: Move beyond answer generation by letting the model use explicit, safe software capabilities.
Activities
•	Define at least two tools/functions with purpose, input schema, output schema, authorization and failure behaviour.
•	Implement tool/function calling through the application/orchestration layer.
•	At least one tool should retrieve current application data or perform a low-risk simulated side effect such as creating a ticket/draft record.
•	Test missing parameters, unauthorized requests, unavailable services and unexpected tool responses.
•	Add human approval before any higher-impact action.
Deliverables
•	Tool Catalogue and tool/API schemas.
•	Working demonstration of at least two tools.
•	Failure/authorization test evidence.
•	Updated architecture diagram.
•	Week 4 progress report (1-2 pages).
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 10: “How Do Autonomous Coding Agents Work?”, pp. 188-192; “Challenges and Limitations” and best practices, pp. 199-204 (coding-agent examples, used here for tool/action thinking).
•	Huyen, AI Engineering: Ch. 6: “Agent Overview” and “Tools,” pp. 275-281.
•	Lanham, AI Agents in Action: Ch. 5: “Defining agent actions” and “Executing ... functions,” pp. 99-107; skim the remaining chapter for semantic/native function patterns.
WEEK 5 (28th Sept-2nd Oct 2026): Agent Architecture and Bounded Autonomy
Weekly focus: Implement one goal-directed, multi-step workflow in which the system chooses among approved next actions and stops safely.
Activities
•	Define one task that genuinely benefits from multi-step decision making.
•	Design Sense/Context -> Plan/Decide -> Act/Tool -> Observe -> Stop/Re-plan.
•	Set maximum iterations, approved tools, stop conditions and human hand-off/approval conditions.
•	Implement the workflow using direct orchestration or a framework.
•	Capture at least three execution traces, including one failure/recovery case.
Deliverables
•	Agent Architecture Diagram.
•	Agent Task Contract: goal, tools, state, limits and stop conditions.
•	Working bounded agent workflow.
•	Three execution traces.
•	Week 5 progress report (1-2 pages).
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 10: “Autonomous Background Coding Agents,” pp. 187-204, focusing on operation, challenges and best practices.
•	Huyen, AI Engineering: Ch. 6: “Planning,” pp. 281-298; “Agent Failure Modes and Evaluation,” pp. 298-300.
•	Lanham, AI Agents in Action: Ch. 11: “Planning: The essential tool,” sequential planning and planner construction, pp. 273-288.
WEEK 6 (5th Oct-9th Oct 2026): Memory, State and Interoperability
Weekly focus: Make the workflow state explicit, retain only justified memory, and connect capabilities through clear interfaces.
Activities
•	Model workflow/session state explicitly.
•	Implement one justified persistent-memory use case (e.g., approved preference, case history or prior task result).
•	Document what is stored, why, who can access it, retention and deletion.
•	Demonstrate that memory improves a legitimate task without silently controlling critical decisions.
•	Implement one external integration OR document one project capability as an MCP-style interface: capability, inputs, outputs, permissions and security boundary.
Deliverables
•	State model.
•	Memory Design and Data Handling Note.
•	Working memory/state demonstration.
•	Integration or MCP-style interface specification.
•	Week 6 progress report (1-2 pages).
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 10: “Challenges and Limitations” / “Best Practices,” pp. 199-204; Ch. 11: “How Autonomous Agents Could Change Software Engineering,” pp. 212-216 (supplementary; agent memory is not a standalone topic in this book).
•	Huyen, AI Engineering: Ch. 6: “Memory,” pp. 300-305; Ch. 10: “AI Pipeline Orchestration,” pp. 472-474.
•	Lanham, AI Agents in Action: Ch. 8: “Implementing memory in agentic systems” and memory/knowledge compression, pp. 200-209.
Reading note on MCP — The three shared books explain tools, memory and orchestration, but do not provide a substantive treatment of the Model Context Protocol (MCP) in the editions supplied. MCP-specific material will therefore come from your personal reading elsewhere; You  are not expected to infer it from these books.

WEEK 7 (12th Oct-16th Oct 2026): Evaluation, Observability and Guardrails
Weekly focus: Produce evidence that the system is sufficiently reliable, grounded, inspectable and safe for its bounded use case.
Activities
•	Create a final evaluation set of at least 30 scenarios: normal, edge, incorrect-information, adversarial, tool-failure and unauthorized-action cases.
•	Define measurable criteria such as task completion, groundedness, tool selection, instruction following, latency and safety violations.
•	Add logs/traces that capture useful model, prompt, retrieval, tool, latency/error and outcome evidence.
•	Implement input/output validation, tool allow-list, authorization, agent iteration limits, approval controls and prompt-injection awareness.
•	Create a Failure Catalogue with at least five genuine failures and re-test after fixes.
Deliverables
•	30+ scenario evaluation dataset and results.
•	Observability/tracing evidence.
•	Guardrail implementation evidence.
•	Failure Catalogue with remediation and re-test evidence.
•	Week 7 progress report (maximum 2 pages).
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 8: testing, maintainability, code review and reliable deployment, especially pp. 153-170; Ch. 9: “Golden Rules for Responsible AI Use,” pp. 181-186.
•	Huyen, AI Engineering: Ch. 4: “Design Your Evaluation Pipeline,” pp. 200-208; Ch. 10: “Put in Guardrails,” pp. 451-455; “Monitoring and Observability,” pp. 465-472.
•	Lanham, AI Agents in Action: Ch. 10: “Employing evaluation for consistent solutions,” pp. 261-270. Optional: Ch. 9 grounding/rubric evaluation, pp. 224-242.
WEEK 8 (19th Oct-23rd Oct 2026):: Hardening, Final Release and Demonstration
Weekly focus: Consolidate the product into a reproducible release and demonstrate value, evidence, limitations and professional ownership.
Activities
•	Review requirements, architecture, model/prompt behaviour, RAG, tools, agent limits, memory, tests, security and documentation.
•	Fix critical defects; document known limitations that remain.
•	Prepare a deployed or reproducibly runnable tagged release.
•	Complete the 8-12 page final engineering report and evidence appendices.
•	Prepare the final live presentation/demonstration.
Deliverables
•	Final tagged release and README/run instructions.
•	Final engineering report (8-12 pages, excluding appendices).
•	Final evaluation/evidence pack.
•	Presentation deck and live demonstration.
•	Completed ClickUp board and final repository evidence.
Required reading for the week
•	Osmani, Beyond Vibe Coding: Ch. 5: “Understanding Generated Code: Review, Refine, Own,” pp. 97-106; Ch. 8: maintainability, code review and reliable deployment, pp. 160-170.
•	Huyen, AI Engineering: Ch. 10: “AI Engineering Architecture,” pp. 449-474, with emphasis on context, guardrails, routing/gateways, agent patterns, monitoring and orchestration.
•	Lanham, AI Agents in Action: Ch. 11: “Applying planning, reasoning, evaluation, and feedback to assistant and agentic systems,” pp. 288-296.
8. Weekly Progress Report (1-2 pages)
The group leader submits one short weekly report with repository and ClickUp evidence. Keep it factual and evidence-based.
•	Group and project name and week ending.
•	Work completed against the weekly objectives.
•	Key engineering decisions and why they were made.
•	Failures/challenges and current response.
•	Links to relevant GitHub commit/tag/PR and ClickUp board/tasks.
•	Individual contribution summary: member, task owned, evidence.
•	Plan for the next week.
9. Final Engineering Report Template (8-12 pages, excluding appendices)
Section	Expected content
1. Executive Summary (about 0.5 page)	Problem, target user, AI-native solution, major result and principal limitation.
2. Problem Context and Requirements (0.75-1 page)	Workflow, requirements, acceptance criteria, AI Boundary Matrix and success measures.
3. System Architecture (about 1 page)	Application architecture; deterministic components; model, RAG, tools, agent, state/memory and external-service boundaries.
4. Foundation Model and Prompt Engineering (0.75-1 page)	Model rationale, prompt architecture/versioning, important iterations and baseline evaluation.
5. Context Engineering and RAG (0.75-1 page)	Corpus/provenance, chunking/indexing, retrieval approach, source grounding and retrieval failures.
6. Tools and Agentic Workflow (1-1.25 pages)	Tool contracts, permissions/side effects, agent goal, plan-act-observe loop, limits and approval/handoff.
7. Memory, State and Interoperability (0.5-0.75 page)	State model, retained memory, privacy/retention and integration/MCP-style interface decisions.
8. Evaluation, Observability and Guardrails (1.5-2 pages)	Evaluation data/criteria/results, genuine failures, traces/logs, guardrails, remediation and re-test evidence.
9. Software Engineering Quality and Deployment (0.75-1 page)	Testing, configuration/secrets, code quality, CI where used, reproducibility/deployment and maintainability.
10. Results, Limitations and Professional Reflection (0.75-1 page)	What works, what does not, remaining risks, cost/latency if relevant, future work and human responsibility.
References	Use a consistent academic style; verify all AI-suggested references.
Appendices (outside page limit)	Detailed user stories, prompt excerpts, test/evaluation tables, tool schemas, logs/traces, AI Engineering Log summary and additional diagrams.

10. Final Presentation and Demonstration Requirements
Each group should prepare a 12-15 minute presentation and live demonstration, followed by questions. The presentation should show the engineered system rather than read the report.
•	Problem, target user and why AI is justified - about 1-2 minutes.
•	Architecture and the progression from model -> RAG -> tools -> bounded agent -> evaluation - about 2 minutes.
•	Live end-to-end successful workflow - about 4 minutes.
•	Show one engineering evidence view: retrieval trace, tool/agent trace, evaluation run or log - about 2 minutes.
•	Show one real failure, limitation or controlled recovery - about 2 minutes.
•	Explain guardrails, human approval boundaries and key lessons - about 1-2 minutes.
Demonstration expectation — A polished happy-path demo is not enough. The team must show that it can inspect what the AI/agent did, explain a real failure, and show where human control is retained.

11. Core Reference Books
1.	Osmani, A. (2025). Beyond Vibe Coding: From Coder to AI-Era Developer. O’Reilly Media.
2.	Huyen, C. (2025). AI Engineering: Building Applications with Foundation Models. O’Reilly Media.
3.	Lanham, M. (2025). AI Agents in Action. Manning Publications.
Closing principle: Build the smallest useful agentic system that you can test, explain and safely control.
