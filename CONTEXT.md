# Teach2SQ

Teach2SQ is an agent education harness for math and physics learning. Its language distinguishes the student's learning artifacts from the agent infrastructure that processes them.

## Language

**Agent Education Harness**:
The operational environment where a teaching model interprets a student query, chooses allowed education tools, preserves evidence, updates memory, and decides the next teaching action.
_Avoid_: Fixed grading pipeline, deterministic MAS, workflow builder

**Assignment Submission**:
A student's homework attempt, including image assets, typed notes, optional reference answer, and optional rubric.
_Avoid_: Prompt, ticket, chat message

**Student Query**:
The student's current request or message to the harness. It may contain a homework attempt, a follow-up question, a quiz answer, or a request for explanation.
_Avoid_: Command, prompt

**Channel Adapter**:
An entrypoint that converts an external channel message, such as dev webhook, Feishu, or WeCom, into a Student Query.
_Avoid_: Bot, integration

**Student**:
The learner who directly interacts with the Teaching Agent.
_Avoid_: User, account

**Teacher**:
The adult or operator who reviews Teaching Logs, audits Student Memory Wiki changes, and adjusts system policy when needed.
_Avoid_: Admin, reviewer

**Teacher Dashboard**:
A lightweight read-only interface for Teachers to inspect students, Live Sessions, Teaching Logs, Trace-linked summaries, memory changes, and low-confidence items.
_Avoid_: Admin backend, grading UI

**Local-Only Access**:
The first-version security assumption that Teacher Dashboard and dev endpoints are reachable only from the developer's machine.
_Avoid_: Authentication, production access control

**Live Session**:
A 24/7 interaction channel where a Student can send Student Queries and receive Teaching Agent responses without per-message Teacher approval.
_Avoid_: Batch run, CLI session

**Local Live Service**:
A developer-run local service that hosts Live Sessions, dev webhooks, and the Teacher Dashboard before production deployment is designed.
_Avoid_: Production deployment, cloud service

**Session Summary**:
A compressed record of recent Live Session context used when the raw conversation becomes too long.
_Avoid_: Student Memory, profile

**Long Task State**:
The persisted state of a Long Task, including its goal, progress, evidence, pending clarification, and recovery information.
_Avoid_: Session Summary, Teaching Log

**Teaching Agent**:
The model-driven agent inside the harness that decides which education tools to call in response to a Student Query.
_Avoid_: Grader, chatbot

**Education Tool**:
A bounded capability the Teaching Agent can call through the tool runtime, such as ingesting an image, reading the Student Memory Wiki, grading an attempt, requesting clarification, updating memory, or generating a Transfer Quiz.
_Avoid_: Pipeline stage, plugin

**Tool Runtime**:
The harness layer that exposes Education Tools to the Teaching Agent, validates inputs and outputs, applies permission gates, and records artifacts.
_Avoid_: Helper functions, service layer

**Agent Runtime**:
The reusable runtime that manages Teaching Agent turns, context assembly, skill loading, tool execution, permissions, artifacts, and recovery for both CLI and Live Session entrypoints.
_Avoid_: CLI app, bot service

**Deterministic MAS**:
A fixed multi-agent workflow where named worker agents are routed by hard-coded control flow instead of letting the Teaching Agent choose tools through the Agent Runtime.
_Avoid_: Architecture target, fallback design

**Teaching Skill**:
A Markdown behavior policy that constrains how the Teaching Agent uses tools for a class of educational situations, and may reference supporting schemas, prompts, tools, and eval fixtures.
_Avoid_: Prompt snippet, action

**Skill Index**:
A concise catalog of available Teaching Skills that helps the Teaching Agent choose which skills to load for a Student Query.
_Avoid_: Prompt list, menu

**Stable Context Prefix**:
The low-churn front portion of an agent turn, such as system contract, foundational skills, skill index, and tool descriptors, arranged to improve prompt cache reuse.
_Avoid_: Full context, memory

**Long Task**:
A multi-step teaching job that may require planning, several tool calls, intermediate artifacts, and recovery across turns.
_Avoid_: Single request, pipeline run

**Image Asset**:
A local image file or IM-provided image URL attached to an Assignment Submission.
_Avoid_: Attachment, upload

**Ingested Problem**:
The structured representation of an Assignment Submission after multimodal transcription, including problem text, formulas, conditions, goals, units, diagrams, and solution steps.
_Avoid_: OCR output, parsed prompt

**Solution Step**:
One semantically meaningful step in the student's attempted solution.
_Avoid_: Line, token, trace

**Grading Result**:
The structured judgment of an Ingested Problem, including correctness, confidence, error locations, explanation, and any clarification request.
_Avoid_: Score, answer

**Clarification Request**:
A request for more information when the system cannot grade reliably because key problem or solution details are ambiguous.
_Avoid_: Error, failure

**Student Memory Wiki**:
A per-student Markdown memory tree that records knowledge state, error patterns, events, and quiz history with evidence links.
_Avoid_: Vector memory, chat history

**Runtime Artifact**:
A structured file produced by the Agent Runtime, such as Long Task State, Session Summary, Trace Log entries, model responses, or tool outputs.
_Avoid_: Student Memory, wiki page

**Model Route**:
A named model-use category in the harness. The first version uses a vision route for multimodal ingestion and DeepSeek for all non-vision routes.
_Avoid_: Provider, model call

**Learning Event**:
A preserved Markdown record of one homework or quiz interaction that can serve as evidence for future memory updates.
_Avoid_: Log entry, transcript

**Teaching Log**:
An audit trail that lets a Teacher review Student Queries, tool calls, model outputs, memory changes, clarifications, and final responses.
_Avoid_: Debug log, terminal output

**Trace Log**:
A developer-facing structured record of model calls, tool calls, schema validation, costs, errors, and runtime decisions.
_Avoid_: Teaching Log, transcript

**Memory Patch**:
A proposed update to the Student Memory Wiki derived from a Grading Result and linked to Learning Events.
_Avoid_: Summary, profile update

**Transfer Quiz**:
One or more follow-up questions generated from the latest error pattern and the Student Memory Wiki.
_Avoid_: Practice set, recommendation

## Example Dialogue

Developer: "The Feishu adapter received a chat message with two photos. Is that already an Assignment Submission?"

Domain expert: "Only after the adapter maps the chat text and image URLs into an Assignment Submission. The raw chat message is outside the core harness language."

Developer: "Does every Student Query become an Assignment Submission?"

Domain expert: "No. A Student Query may be a follow-up question or quiz answer. It becomes an Assignment Submission only when the Teaching Agent identifies homework evidence to inspect."

Developer: "Does the Teacher approve every Teaching Agent response?"

Domain expert: "No. A Live Session should run without per-message approval. The Teacher reviews Teaching Logs and memory changes after the fact."

Developer: "Is the Student Memory Wiki the same thing as the recent chat context?"

Domain expert: "No. Session Summary is short-term interaction context, Long Task State tracks in-progress work, Student Memory Wiki stores durable learning observations, and Teaching Log is the audit trail."

Developer: "Should a Teacher read raw model JSON to audit a student interaction?"

Domain expert: "No. The Teacher reads the Teaching Log. Developers use the Trace Log, and both logs are linked by shared identifiers."

Developer: "Should the Teaching Agent always call the same tools in the same order?"

Domain expert: "No. The Teaching Agent has an open tool space for Long Tasks. Teaching Skills constrain how it behaves in educational situations."

Developer: "Should we create separate grader, memory, quiz, and ingestion agents and route between them deterministically?"

Domain expert: "No. That would be a Deterministic MAS. Teach2SQ is an Agent Education Harness: one Teaching Agent reasons through an Agent Runtime and calls Education Tools as needed."

Developer: "Are all Teaching Skills loaded into every turn?"

Domain expert: "No. Foundational skills are always loaded, the Teaching Agent selects task skills from the Skill Index, and some tools require specific skills before they can run."

Developer: "Qwen returned OCR text. Should we store that as memory?"

Domain expert: "No. Qwen output becomes an Ingested Problem first. Student Memory Wiki pages should be updated only through a Memory Patch with evidence links to the Learning Event."

Developer: "Should Trace Log entries be written into the Student Memory Wiki?"

Domain expert: "No. Runtime Artifacts are structured operational records. The Student Memory Wiki stores durable learning observations."

Developer: "DeepSeek is unsure whether a formula says `m=5kg` or `m=5g`. Should we grade anyway?"

Domain expert: "No. That should become a Clarification Request because the ambiguity can change the physics result."
