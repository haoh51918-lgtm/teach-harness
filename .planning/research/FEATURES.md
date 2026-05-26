# Feature Research: Teach2SQ

**Date:** 2026-05-26
**Milestone context:** Greenfield local-first agent education harness

## Table Stakes

### Runtime And Entrypoints

- Shared Agent Runtime used by CLI, dev webhook, and Local Live Service.
- Student Query normalization from CLI and HTTP payloads.
- Long Task State persistence and recovery.
- Session Summary compaction for long live sessions.
- Loop limits and explicit stop conditions.
- Stable Context Prefix plus Dynamic Context Tail.

### Teaching Skills

- Always-loaded foundational skills:
  - `teaching-safety`
  - `student-tone`
  - `memory-safety`
- Task skills:
  - `image-ingestion`
  - `homework-grading`
  - `concept-explanation`
  - `transfer-quiz`
- Skill Index for selection.
- Tool-required skill gates.

### Education Tools

- `read_student_memory`
- `search_student_memory`
- `ingest_image_submission`
- `grade_attempt`
- `explain_concept`
- `propose_memory_patch`
- `apply_memory_patch`
- `record_learning_event`
- `generate_transfer_quiz`
- `ask_clarification`
- `summarize_session`
- `finish_response`

### Tool Runtime

- Tool descriptors with name, description, input schema, output schema, permission level, required skills, artifact behavior, memory-write behavior, external-call behavior, and error shape.
- Argument validation before execution.
- Result validation after execution.
- Permission checks for `read`, `compute`, `write_memory`, `external_effect`, and no first-version `destructive`.
- Tool call trace recording and artifact writing.

### Model Routing

- Named routes for `vision`, `agent`, `grader`, `memory`, `summarizer`, and `quiz`.
- Provider configuration through environment variables.
- Timeout, retry, schema validation, raw response artifact capture, and route-level trace fields.
- Fake and recorded providers for tests.

### Assignment And Grading

- Assignment Submission detection inside Student Queries.
- Image Asset ingestion with local path and URL support.
- Ingested Problem schema including text, formulas, diagrams, conditions, goals, units, solution steps, confidence, and ambiguities.
- Grading Result schema with correctness, rubric support, confidence, error locations, explanation, and optional clarification request.
- Clarification Request path when key evidence is ambiguous.
- Student-facing Markdown feedback and machine-readable JSON artifacts.

### Student Memory Wiki

- Markdown-first per-student wiki.
- `profile.md`, topic knowledge pages, error-pattern pages, event pages, and quiz pages.
- Learning Events as evidence records.
- Memory Patches linked to evidence.
- Pending observations for low-confidence findings.
- Stable observations only after sufficient confidence/evidence.

### Logs And Audit

- Teacher-facing Teaching Logs.
- Developer-facing Trace Logs.
- Links among Teaching Logs, Trace Logs, artifacts, Learning Events, and Memory Patches.
- Failed-turn entries for schema failures, ambiguous evidence, and retry exhaustion.

### Local Live Service

- FastAPI startup and health route.
- `POST /dev/query`.
- Configured image URL download behavior.
- Local artifact serving for dashboard inspection.
- Local-only access assumption.

### Teacher Dashboard

- Student list with recent activity.
- Student detail with profile summary and memory pages.
- Live Session list.
- Teaching Log detail.
- Memory patch/diff and pending observation views.
- Trace links for developers.
- No destructive actions in v1.

### Evaluation

- Algebra/function sample assignments.
- Mechanics sample assignments.
- Ambiguous image/text cases.
- Expected critical facts for ingestion and grading.
- Expected memory updates or pending observations.
- Expected Transfer Quiz misconception targets.
- Recorded/fake model fixtures.

## Differentiators

- Agent harness architecture rather than a deterministic grading chain.
- Evidence-linked Student Memory Wiki readable by Teachers.
- Explicit separation of Session Summary, Long Task State, Runtime Artifacts, Student Memory Wiki, Teaching Logs, and Trace Logs.
- Skill-constrained open tool choice.
- Conservative memory writes with pending/stable classification.
- Local-first 24/7 Live Session simulation before production IM integration.
- Trace-linked read-only Teacher Dashboard.
- Testable anti-regression rule that prevents introducing `teach2sq/agents/` without revisiting ADR-0004.

## Anti-Features

- Per-message Teacher approval.
- Production Feishu/WeCom bot in v1.
- Full admin dashboard or classroom management system.
- Hard-coded workflow like image -> grader -> memory -> quiz.
- Worker-agent packages for grading/memory/quiz/ingestion.
- Raw model output written directly into memory.
- Chat transcript used as durable memory.
- Fake numerical precision when no rubric exists.
- Confident grading when a formula, unit, diagram, or step ambiguity can change the answer.
- Destructive memory editing from student flows.

## Feature Dependencies

- Schemas must precede Tool Runtime, model routing, artifacts, and memory patches.
- Artifact/state storage should precede Tool Runtime and Agent Runtime.
- Skill Loader should precede real tool execution because some tools require skills.
- Tool Runtime and Model Router should precede Core Education Tools.
- Student Memory Wiki should precede memory tools and Teacher Dashboard memory views.
- Agent Runtime Loop should follow schemas, state, skills, tools, and model routing.
- CLI and service should call the shared runtime after it exists.
- Teacher Dashboard depends on logs, memory pages, artifacts, and trace IDs.
- Evaluation harness should start after enough schemas/tools exist, then grow with each phase.
