# Requirements: Teach2SQ

**Defined:** 2026-05-26
**Core Value:** Students receive reliable, age-appropriate math and physics help while every teaching action, model judgment, artifact, and durable memory update remains evidence-linked and auditable by a Teacher.

## v1 Requirements

### Project Foundation

- [ ] **FOUND-01**: Developer can install the project dependencies and import the `teach2sq` package without provider credentials.
- [ ] **FOUND-02**: Developer can run a Typer CLI `--help` command without invoking agent behavior.
- [ ] **FOUND-03**: Developer can start a FastAPI app and call a health route without invoking agent behavior.
- [ ] **FOUND-04**: Repository contains package, tests, examples, runtime, and students directories with clear boundaries matching the Teach2SQ domain language.

### Schemas

- [ ] **SCHEMA-01**: Developer can validate a Student Query with student ID, session ID, channel, text, image assets, timestamp, and optional task ID.
- [ ] **SCHEMA-02**: Developer can validate image assets from local paths or URLs with source metadata and artifact references.
- [ ] **SCHEMA-03**: Developer can validate Assignment Submission, Ingested Problem, Solution Step, Grading Result, Clarification Request, and Quiz Set contracts.
- [ ] **SCHEMA-04**: Developer can validate Agent Turn, Tool Descriptor, Tool Call Record, Teaching Log Entry, Trace Log Entry, Long Task State, Memory Patch, and Learning Event contracts.
- [ ] **SCHEMA-05**: Developer can export JSON Schema for all public contracts.
- [ ] **SCHEMA-06**: Invalid model-output states that could affect grading, memory, or quiz generation fail validation instead of being silently accepted.

### Artifacts And State

- [ ] **STATE-01**: Runtime can create deterministic artifact IDs and paths for JSON, JSONL, Markdown, and binary artifacts.
- [ ] **STATE-02**: Runtime can append Teaching Log and Trace Log records while preserving links to artifact IDs and turn IDs.
- [ ] **STATE-03**: Runtime can persist and reload Session Summary records for long live sessions.
- [ ] **STATE-04**: Runtime can persist and reload Long Task State with goal, progress, evidence, pending clarification, completed tool calls, and recovery hints.
- [ ] **STATE-05**: Runtime records corrupt-file or schema-load failures as explicit errors instead of losing state silently.

### Teaching Skills

- [ ] **SKILL-01**: Runtime always loads foundational Teaching Skills for teaching safety, student tone, and memory safety.
- [ ] **SKILL-02**: Runtime exposes a Skill Index that lets the Teaching Agent select task skills by relevance.
- [ ] **SKILL-03**: Runtime can load task Teaching Skills for image ingestion, homework grading, concept explanation, and transfer quiz generation.
- [ ] **SKILL-04**: Tool Runtime rejects tool calls when required Teaching Skills are missing.
- [ ] **SKILL-05**: Stable Context Prefix assembly includes system contract, foundational skills, Skill Index, and stable tool descriptors.

### Tool Runtime

- [ ] **TOOL-01**: Developer can register Education Tools with name, description, input schema, output schema, permission level, required skills, artifact behavior, memory-write behavior, external-call behavior, and error shape.
- [ ] **TOOL-02**: Tool Runtime validates tool arguments before execution and tool results after execution.
- [ ] **TOOL-03**: Tool Runtime enforces permission levels for read, compute, write_memory, external_effect, and denies destructive operations in student flows.
- [ ] **TOOL-04**: Tool Runtime writes Tool Call Records and Trace Log entries for successful, failed, denied, and validation-error calls.
- [ ] **TOOL-05**: Architecture checks fail if a `teach2sq/agents/` package is introduced without revising ADR-0004.

### Model Routing

- [ ] **MODEL-01**: Runtime exposes named model routes for vision, agent, grader, memory, summarizer, and quiz.
- [ ] **MODEL-02**: Runtime loads exact provider model IDs and credentials from environment or config without hard-coding them into tool helpers.
- [ ] **MODEL-03**: Runtime can call fake or recorded model providers during automated tests.
- [ ] **MODEL-04**: Runtime records route, provider model ID, latency, retry count, schema validation status, token usage when available, and raw response artifact path for each model call.
- [ ] **MODEL-05**: Runtime retries bounded schema failures and emits a failed turn when repair is exhausted.
- [ ] **MODEL-06**: Vision route normalizes local image paths and image URLs into provider-supported requests before calling Qwen/VL-compatible models.

### Student Memory

- [ ] **MEM-01**: Runtime can create a Markdown-first Student Memory Wiki with profile, knowledge, error-pattern, event, and quiz pages.
- [ ] **MEM-02**: Runtime can write Learning Event pages that preserve assignment or quiz evidence and links to runtime artifacts.
- [ ] **MEM-03**: Runtime can validate Memory Patches with target pages, proposed changes, confidence, evidence links, and pending or stable classification.
- [ ] **MEM-04**: Runtime prevents stable memory updates when evidence is missing or confidence is too low.
- [ ] **MEM-05**: Runtime can read and search Student Memory Wiki pages for context assembly and Education Tools.
- [ ] **MEM-06**: Runtime keeps Student Memory Wiki separate from Session Summary, Long Task State, Teaching Logs, and Trace Logs.

### Education Tools

- [ ] **EDU-01**: Teaching Agent can call `read_student_memory` and `search_student_memory` through Tool Runtime.
- [ ] **EDU-02**: Teaching Agent can call `ingest_image_submission` to create an Ingested Problem from image assets and student text.
- [ ] **EDU-03**: Teaching Agent can call `grade_attempt` to produce a Grading Result with correctness, confidence, error locations, explanation, and optional rubric score.
- [ ] **EDU-04**: Teaching Agent can call `ask_clarification` when ambiguity in formula, unit, diagram, or solution step could change the answer.
- [ ] **EDU-05**: Teaching Agent can call `explain_concept` to produce age-appropriate Markdown explanation without requiring an assignment.
- [ ] **EDU-06**: Teaching Agent can call `record_learning_event` to preserve evidence from homework or quiz interactions.
- [ ] **EDU-07**: Teaching Agent can call `propose_memory_patch` and `apply_memory_patch` only when evidence-linked memory safety rules pass.
- [ ] **EDU-08**: Teaching Agent can call `generate_transfer_quiz` to create one to three follow-up questions targeting a recent error pattern.
- [ ] **EDU-09**: Teaching Agent can call `summarize_session` and `finish_response` to compact context and return student-readable Markdown.

### Agent Runtime Loop

- [ ] **AGENT-01**: Runtime assembles Stable Context Prefix and Dynamic Context Tail for each Student Query.
- [ ] **AGENT-02**: Teaching Agent can decide to answer directly, load a skill, call an Education Tool, request clarification, update state, or finish.
- [ ] **AGENT-03**: Runtime executes selected tool calls only through Tool Runtime.
- [ ] **AGENT-04**: Runtime stops safely on final response, clarification request, loop limit, validation failure, or recoverable task state.
- [ ] **AGENT-05**: Runtime can recover an in-progress Long Task from persisted state across turns.
- [ ] **AGENT-06**: Tests prove the same query type can lead to different tool sequences depending on context rather than a fixed workflow.

### Entrypoints And Service

- [ ] **ENTRY-01**: Developer can submit a Student Query through `teach2sq query` with text, student ID, session ID, optional task ID, and optional image path.
- [ ] **ENTRY-02**: CLI response reports final student Markdown plus artifact, Teaching Log, and Trace Log references.
- [ ] **ENTRY-03**: Developer can submit a Student Query through `POST /dev/query`.
- [ ] **ENTRY-04**: Local Live Service submits webhook requests to the shared Agent Runtime and returns final response plus log/artifact IDs.
- [ ] **ENTRY-05**: Local Live Service can download configured image URLs as logged external effects.
- [ ] **ENTRY-06**: Feishu and WeCom adapter skeletons convert sample payloads into Student Query objects without implementing production auth.

### Teacher Dashboard

- [ ] **DASH-01**: Teacher can view a local-only student list with recent activity.
- [ ] **DASH-02**: Teacher can view a student detail page with profile summary and memory pages.
- [ ] **DASH-03**: Teacher can view Live Session list and Teaching Log detail pages.
- [ ] **DASH-04**: Teacher can inspect memory patch diffs and low-confidence or pending observations.
- [ ] **DASH-05**: Teacher can follow links from Teaching Logs to developer-facing Trace Log IDs and artifacts.
- [ ] **DASH-06**: Dashboard exposes no destructive actions in v1.

### Evaluation And Documentation

- [ ] **EVAL-01**: Developer can run recorded/fake-provider evals for algebra, functions, and mechanics assignments.
- [ ] **EVAL-02**: Evaluation cases include ambiguous image or text inputs that must trigger clarification instead of confident grading.
- [ ] **EVAL-03**: Evaluation cases check expected critical facts for ingestion and grading.
- [ ] **EVAL-04**: Evaluation cases check expected memory updates, pending observations, or Transfer Quiz misconception targets.
- [ ] **DOC-01**: README explains setup, environment variables, local run commands, and local-only security assumptions.
- [ ] **DOC-02**: Documentation explains how to add a Teaching Skill, add an Education Tool, read Teaching Logs, and read Trace Logs.

## v2 Requirements

### Production Integrations

- **PROD-01**: Deploy production Feishu or WeCom bot with platform authentication and security review.
- **PROD-02**: Add production authentication and access control for Teacher Dashboard.
- **PROD-03**: Add durable production storage beyond local filesystem if needed.
- **PROD-04**: Add deployment packaging such as Docker only after local harness behavior is validated.

### Expanded Product Scope

- **CLASS-01**: Support multi-tenant classroom management.
- **CURR-01**: Add full curriculum graph and long-term learning path planning.
- **MATH-01**: Add CAS-level symbolic proof support.
- **ADMIN-01**: Add controlled teacher actions for memory review if read-only dashboard is insufficient.

## Out of Scope

| Feature | Reason |
|---------|--------|
| Fixed grading pipeline | Violates harness architecture and ADR-0004 |
| Deterministic MAS with grader/memory/quiz/ingestion workers | Replaces Teaching Agent tool choice with hard-coded routing |
| Production Feishu/WeCom deployment | V1 validates local harness and adapter boundaries first |
| Teacher login or production access control | V1 assumes localhost-only dashboard and dev endpoints |
| Docker/cloud deployment/queueing | V1 is a developer-run local service |
| Multi-tenant classroom management | Not needed to validate core education harness |
| Destructive memory editing from student flows | Unsafe for evidence-linked learning memory |
| Per-message Teacher approval | Teachers audit logs and memory changes after the fact |
| CAS-level symbolic proof | Out of first educational scope |
| Full curriculum graph | Out of first educational scope |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01 | Unmapped | Pending |
| FOUND-02 | Unmapped | Pending |
| FOUND-03 | Unmapped | Pending |
| FOUND-04 | Unmapped | Pending |
| SCHEMA-01 | Unmapped | Pending |
| SCHEMA-02 | Unmapped | Pending |
| SCHEMA-03 | Unmapped | Pending |
| SCHEMA-04 | Unmapped | Pending |
| SCHEMA-05 | Unmapped | Pending |
| SCHEMA-06 | Unmapped | Pending |
| STATE-01 | Unmapped | Pending |
| STATE-02 | Unmapped | Pending |
| STATE-03 | Unmapped | Pending |
| STATE-04 | Unmapped | Pending |
| STATE-05 | Unmapped | Pending |
| SKILL-01 | Unmapped | Pending |
| SKILL-02 | Unmapped | Pending |
| SKILL-03 | Unmapped | Pending |
| SKILL-04 | Unmapped | Pending |
| SKILL-05 | Unmapped | Pending |
| TOOL-01 | Unmapped | Pending |
| TOOL-02 | Unmapped | Pending |
| TOOL-03 | Unmapped | Pending |
| TOOL-04 | Unmapped | Pending |
| TOOL-05 | Unmapped | Pending |
| MODEL-01 | Unmapped | Pending |
| MODEL-02 | Unmapped | Pending |
| MODEL-03 | Unmapped | Pending |
| MODEL-04 | Unmapped | Pending |
| MODEL-05 | Unmapped | Pending |
| MODEL-06 | Unmapped | Pending |
| MEM-01 | Unmapped | Pending |
| MEM-02 | Unmapped | Pending |
| MEM-03 | Unmapped | Pending |
| MEM-04 | Unmapped | Pending |
| MEM-05 | Unmapped | Pending |
| MEM-06 | Unmapped | Pending |
| EDU-01 | Unmapped | Pending |
| EDU-02 | Unmapped | Pending |
| EDU-03 | Unmapped | Pending |
| EDU-04 | Unmapped | Pending |
| EDU-05 | Unmapped | Pending |
| EDU-06 | Unmapped | Pending |
| EDU-07 | Unmapped | Pending |
| EDU-08 | Unmapped | Pending |
| EDU-09 | Unmapped | Pending |
| AGENT-01 | Unmapped | Pending |
| AGENT-02 | Unmapped | Pending |
| AGENT-03 | Unmapped | Pending |
| AGENT-04 | Unmapped | Pending |
| AGENT-05 | Unmapped | Pending |
| AGENT-06 | Unmapped | Pending |
| ENTRY-01 | Unmapped | Pending |
| ENTRY-02 | Unmapped | Pending |
| ENTRY-03 | Unmapped | Pending |
| ENTRY-04 | Unmapped | Pending |
| ENTRY-05 | Unmapped | Pending |
| ENTRY-06 | Unmapped | Pending |
| DASH-01 | Unmapped | Pending |
| DASH-02 | Unmapped | Pending |
| DASH-03 | Unmapped | Pending |
| DASH-04 | Unmapped | Pending |
| DASH-05 | Unmapped | Pending |
| DASH-06 | Unmapped | Pending |
| EVAL-01 | Unmapped | Pending |
| EVAL-02 | Unmapped | Pending |
| EVAL-03 | Unmapped | Pending |
| EVAL-04 | Unmapped | Pending |
| DOC-01 | Unmapped | Pending |
| DOC-02 | Unmapped | Pending |

**Coverage:**
- v1 requirements: 68 total
- Mapped to phases: 0
- Unmapped: 68

---
*Requirements defined: 2026-05-26*
*Last updated: 2026-05-26 after initial definition*
