# Sentinel
Autonomous workflows with a human safety net
What Inspired Us, What We Learned, and How We Built It
We were inspired by a simple observation: organizations do not suffer from a shortage of AI-generated text—they suffer from a shortage of reliable action. A copilot can draft a report, but a public-service officer, clinician, supply-chain planner, or bank operations team still needs a system that understands context, verifies facts, obtains approvals, executes workflows, and leaves an accountable trail. The Applied AI & Agentic Intelligence theme—especially its emphasis on governance, safety, observability, and human oversight—motivated us to build beyond a chatbot and toward a governable multi-agent system. The 11th TiE Global Summit’s Applied AI Arena explicitly highlights “Agentic Workflows & Enterprise Infrastructure,” which reinforced that practical orchestration, not just model capability, is the core challenge.
The Problem We Chose
We focused on citizen-service case resolution as our vertical: a domain where requests are high-volume, rules change, documents are unstructured, and mistakes can affect real people.
Traditional systems often behave like static rule engines:
\text{Request} \rightarrow \text{Rules} \rightarrow \text{Decision}
That model breaks when a case has missing evidence, conflicting regulations, an unusual exception, or a need for coordination across departments. Our goal was to replace that brittle pipeline with an adaptive, auditable workflow:
\text{Perceive} \rightarrow \text{Plan} \rightarrow \text{Verify} \rightarrow \text{Act} \rightarrow \text{Observe} \rightarrow \text{Escalate}
The system should not merely “answer” a citizen query. It should classify the request, retrieve the correct policy, identify missing documents, route work to the right team, generate a draft decision with citations, request human approval when risk is high, and log every step.
What Inspired Us
Three tensions inspired the project:
•	Automation versus accountability. Autonomous agents can accelerate decisions, but public systems need traceability, permissions, and human review.
•	General intelligence versus domain reliability. A general-purpose model may be fluent, but a high-stakes domain needs structured policies, validation rules, and evidence-grounded reasoning.
•	Individual agents versus organizational workflows. A single agent cannot safely manage procurement, compliance, citizen support, finance, and operations. Specialized agents need shared protocols, feedback loops, and clear ownership.
We also saw that agent sprawl is a real risk. If every team creates its own autonomous assistant with broad tool access, organizations can quickly lose control over who can do what, why an action occurred, and how to reverse it. Our design therefore treats governance as a first-class product feature—not an afterthought.
What We Learned
Agents Need Contracts
The biggest lesson was that agents should not communicate through vague natural-language messages alone. We gave each agent a narrow role, a typed task contract, allowed tools, escalation conditions, and a required output schema.
For example, a document-intake agent returns:
{
  "case_id": "CS-2048",
  "request_type": "benefit_renewal",
  "documents_found": ["income_proof.pdf"],
  "missing_documents": ["address_proof.pdf"],
  "confidence": 0.86,
  "next_action": "request_missing_documents"
}
This made handoffs predictable. The orchestration layer could validate the response, route the case, and prevent an agent from inventing a field or taking an unauthorized action.
Planning Must Be Constrained
We initially let the planner generate long, free-form plans. It was impressive in demos but unreliable under edge cases. We learned to constrain planning with a finite state machine:
S = \{\text{Intake}, \text{Evidence}, \text{Policy}, \text{Review}, \text{Approval}, \text{Execute}, \text{Close}\}
The planner proposes the next state and required actions, but the workflow engine enforces valid transitions. This gave us adaptability without sacrificing control.
Confidence Is Not Truth
A model saying it is “90% confident” is not the same as proving a claim. We separated:
•	Retrieval confidence: Did we find relevant policy and records?
•	Extraction confidence: Did we correctly identify fields from documents?
•	Rule compliance: Does the case satisfy explicit business rules?
•	Human approval: Is the action reversible, sensitive, or high-impact?
A decision could only proceed when the required checks passed. Otherwise, the system created a human-review task rather than silently guessing.
How We Built the Project
System Architecture
We built AegisFlow, a governable multi-agent workflow platform for citizen-service operations.
Layer	Purpose	Example
Experience layer	Citizen/officer interface, case timeline, approvals	Officer dashboard and citizen status portal
Orchestration layer	Task routing, retries, state management, escalation	Workflow engine and agent supervisor
Agent layer	Specialized reasoning and execution	Intake, policy, fraud, scheduling, and communication agents
Tool layer	Controlled access to systems and data	CRM, document store, payment gateway, calendar, notification service
Governance layer	Permissions, audit trail, policy guardrails, human-in-the-loop	Approval policies and immutable event log
The central principle was least privilege: each agent could access only the tools and data fields necessary for its role. For instance, the communication agent could send notifications but could not modify eligibility decisions or access raw financial records.
Agent Roles
•	Intake Agent: Parses requests, classifies intent, extracts entities, and identifies missing information.
•	Policy Agent: Retrieves applicable rules, regulations, and precedents with citations.
•	Verification Agent: Cross-checks documents, data fields, and business rules.
•	Risk Agent: Flags anomalies, potential fraud, conflicts of interest, and high-impact actions.
•	Workflow Agent: Assigns tasks, schedules follow-ups, and coordinates department handoffs.
•	Approval Agent: Prepares a concise decision package for a human reviewer.
•	Audit Agent: Records prompts, tool calls, retrieved evidence, decisions, approvals, and outcomes.
Reasoning and Verification
For every recommendation, the system produced a structured rationale:
\text{Decision} = f(\text{Citizen context}, \text{Policy evidence}, \text{Verification results}, \text{Risk score})
Instead of presenting a black-box answer, the UI displayed:
•	The policy clauses used.
•	The documents and database records referenced.
•	The rules that passed or failed.
•	The uncertainty and missing evidence.
•	The proposed action and its reversibility.
•	The human approval required, if any.
This turned AI output into an evidence-backed decision package, which was far more useful to an officer than a paragraph of generated text.
Human-in-the-Loop Controls
We used a risk-based approval model. Low-risk actions, such as sending a status update, could proceed automatically. High-risk actions, such as rejecting an application, changing eligibility, issuing a payment, or accessing sensitive records, required explicit human approval.
We defined an action-risk score:
R = w_1I + w_2P + w_3U + w_4F
where I is impact, P is policy sensitivity, U is uncertainty, and F is financial or legal exposure. When R exceeded a configured threshold, the workflow paused for review.
This made autonomy configurable: organizations could start with conservative oversight and gradually expand automation as trust, evaluation results, and audit data matured.
Challenges We Faced
Ambiguous Real-World Requests
Citizens rarely describe problems in clean categories. One request might combine a renewal, an address change, a missing document, and an urgent deadline. Our first classifier often forced cases into a single category.
We solved this by allowing multiple intents, storing unresolved ambiguities as clarification tasks, and requiring the intake agent to ask targeted questions instead of guessing.
Tool Errors and Cascading Failures
During testing, an external API timeout caused one agent to retry repeatedly, which created duplicate notifications. That taught us that autonomy without circuit breakers is dangerous.
We added:
•	Idempotency keys for irreversible actions.
•	Retry limits with exponential backoff.
•	Dead-letter queues for failed tasks.
•	Automatic escalation after repeated failure.
•	A “pause all agents” control for incident response.
Keeping the Audit Trail Useful
Logging everything is easy; making logs understandable is hard. Early versions produced thousands of low-value events. We redesigned observability around a case timeline: each entry showed the agent, action, input evidence, policy reference, output, approval status, and resulting state change.
The goal was not merely to record what happened, but to let an auditor answer: “Why did the system make this decision, and who approved it?”
Avoiding Over-Engineering
With limited hackathon time, we wanted to build every possible agent. We learned to prioritize one end-to-end journey: intake, verification, policy retrieval, draft decision, human approval, and notification. A smaller but complete workflow demonstrated more value than many disconnected AI features.
What We Would Build Next
For a production-ready version, we would add:
•	A policy simulator to test rule changes before deployment.
•	Red-team evaluations for prompt injection, data leakage, and unsafe tool use.
•	Role-based access control integrated with enterprise identity systems.
•	Continuous evaluation using labeled historical cases and expert reviewer feedback.
•	A feedback loop where human corrections improve retrieval, extraction, and routing over time.
•	Multi-tenant deployment so different government departments or enterprises can configure their own policies, tools, and approval thresholds.
Why It Matters
Our project showed that the future of applied AI is not “more autonomous agents.” It is accountable autonomy: systems that can act across complex workflows, but only within defined permissions, verifiable evidence, observable logs, and meaningful human control.
By 2036, the most valuable AI systems will not simply generate content. They will coordinate specialized agents, reason over domain knowledge, execute multi-step work safely, and give humans the confidence to delegate decisions without surrendering responsibility. That is the future we set out to build.

