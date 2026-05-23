# Teach2SQ Agent Runtime Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a local Claude Code-like education harness where students interact with a Teaching Agent, the agent selects Teaching Skills and Education Tools, teachers inspect logs through a read-only dashboard, and all durable learning memory is evidence-linked.

**Architecture:** Teach2SQ is a shared Agent Runtime, not a fixed grading pipeline. CLI, dev webhook, and Teacher Dashboard all call the same runtime; the runtime assembles context, loads skills, routes model calls, dispatches tools, applies permission gates, records artifacts, and persists memory/log state. The first implementation runs as a Local Live Service with local-only access.

**Tech Stack:** Python, FastAPI, Typer, Pydantic, DeepSeek for non-vision model routes, Qwen/VL for vision ingestion, Markdown Student Memory Wiki, JSON/JSONL Runtime Artifacts, lightweight server-rendered or minimal TypeScript Teacher Dashboard.

---

## Reference Principles

This plan intentionally follows the harness ideas from the referenced Claude Code learning projects:

- Agent behavior is driven by a loop: assemble context, call model, dispatch tools, record results, and decide whether to continue.
- Most product complexity belongs in deterministic infrastructure: context assembly, tool descriptors, permission gates, artifacts, traces, state recovery, and compaction.
- Skills are behavior policies loaded into the agent context. They constrain tool use and teaching behavior without turning the system into a hard-coded pipeline.
- Tools are explicit capabilities with schemas, permissions, execution records, and artifacts. They are not informal helper functions.
- Long tasks need persisted state and recovery. The runtime should not assume a single request finishes every teaching job.

Reference links:

- `shareAI-lab/learn-claude-code`: https://github.com/shareAI-lab/learn-claude-code/tree/main
- `VILA-Lab/Dive-into-Claude-Code`: https://github.com/VILA-Lab/Dive-into-Claude-Code

## Superseded Document

This plan supersedes `docs/superpowers/specs/2026-05-23-teach2sq-harness-design.md`.

The old document was CLI-first and pipeline-shaped. The current direction is:

- Agent Runtime first, with CLI and Local Live Service as entrypoints.
- Open tool choice constrained by Teaching Skills.
- Local 7x24 student interaction via dev webhook before real Feishu/WeCom deployment.
- Teacher Dashboard in first version as a local-only read-only view.
- Student Memory Wiki remains Markdown-first; runtime state and traces are structured artifacts.

## Domain Language

Canonical terms live in `CONTEXT.md`. Implementation and docs should use these names consistently:

- Student Query
- Teaching Agent
- Agent Runtime
- Teaching Skill
- Skill Index
- Education Tool
- Tool Runtime
- Student Memory Wiki
- Learning Event
- Teaching Log
- Trace Log
- Runtime Artifact
- Long Task State
- Session Summary
- Channel Adapter
- Teacher Dashboard
- Local Live Service

## Target System Shape

```text
                 +-------------------+
Student / Dev -> | Channel Adapter   | -- Student Query -->
                 +-------------------+
                           |
                           v
                 +-------------------+
                 | Agent Runtime     |
                 | - context builder |
                 | - skill loader    |
                 | - model router    |
                 | - tool runtime    |
                 | - permissions     |
                 | - recovery        |
                 +-------------------+
                   |       |       |
                   v       v       v
              Skills    Tools   Artifacts
                   |       |       |
                   v       v       v
             Memory Wiki  Logs  Task State
                           |
                           v
                 +-------------------+
Teacher -------> | Teacher Dashboard |
                 +-------------------+
```

## Runtime Loop

Each Student Query enters the shared Agent Runtime:

1. Normalize the incoming message into a Student Query.
2. Load Stable Context Prefix: system contract, foundational Teaching Skills, Skill Index, and stable tool descriptors.
3. Load Dynamic Context Tail: current query, session summary, relevant Student Memory Wiki excerpts, Long Task State, recent Teaching Log summary, and image metadata.
4. Ask DeepSeek to choose the next action: answer directly, load a skill, call an Education Tool, request clarification, update state, or finish.
5. Tool Runtime validates arguments, checks permissions, executes deterministic code, writes Runtime Artifacts, and returns a structured result.
6. Runtime appends Trace Log and Teaching Log entries.
7. Runtime repeats until the agent finishes, reaches a configured loop limit, or needs student clarification.
8. Runtime writes final student response, updates Session Summary or Long Task State, and exposes the turn to the Teacher Dashboard.

The agent may choose tools in an open-ended way, but all tools are bounded by descriptors, schemas, permissions, skills, and artifact contracts.

## State Model

The first version has four distinct state layers:

| Layer | Purpose | Storage |
| --- | --- | --- |
| Session Summary | Recent Live Session context after compaction | `runtime/sessions/<session_id>/summary.md` and JSON metadata |
| Long Task State | In-progress multi-step teaching jobs | `runtime/tasks/<task_id>.json` plus readable Markdown summary |
| Student Memory Wiki | Durable learning observations and evidence links | `students/<student_id>/**/*.md` |
| Teaching/Trace Logs | Teacher audit and developer replay | `runtime/logs/teaching/*.md` and `runtime/logs/traces/*.jsonl` |

Student Memory Wiki is not a dump of chat history. It only receives Memory Patches linked to Learning Events.

Runtime Artifacts are not Student Memory. They are operational evidence, model outputs, tool outputs, traces, and task state.

## File Structure To Create

```text
CONTEXT.md
docs/
  adr/
    0001-open-agent-loop-with-teaching-skills.md
    0002-python-core-tools-with-mcp-compatible-contract.md
    0003-shared-agent-runtime-for-cli-and-live-sessions.md
  superpowers/
    plans/
      2026-05-24-teach2sq-agent-runtime-plan.md
teach2sq/
  __init__.py
  app/
    server.py
    routes_dev_webhook.py
    routes_teacher_dashboard.py
  runtime/
    agent_loop.py
    context_builder.py
    skill_loader.py
    model_router.py
    permissions.py
    artifacts.py
    state_store.py
  tools/
    registry.py
    descriptors.py
    memory_tools.py
    vision_tools.py
    grading_tools.py
    quiz_tools.py
    logging_tools.py
  schemas/
    queries.py
    tools.py
    memory.py
    logs.py
    tasks.py
  skills/
    teaching-safety/SKILL.md
    student-tone/SKILL.md
    memory-safety/SKILL.md
    image-ingestion/SKILL.md
    homework-grading/SKILL.md
    concept-explanation/SKILL.md
    transfer-quiz/SKILL.md
  memory/
    wiki.py
    patches.py
  cli.py
tests/
  runtime/
  tools/
  memory/
  app/
examples/
  assignments/
  images/
runtime/
  .gitkeep
students/
  .gitkeep
```

Implementation may adjust names slightly, but it should preserve these boundaries.

## Core Schemas

The plan should define Pydantic models for:

- `StudentQuery`: student ID, session ID, channel, text, image assets, query timestamp, optional task ID.
- `ImageAsset`: local path or URL, source channel, MIME type when known, artifact reference when downloaded.
- `AgentTurn`: query, selected skills, loaded memory excerpts, tool calls, final response, task status.
- `ToolDescriptor`: name, description, input schema, output schema, permission level, required skills, artifact contract.
- `ToolCallRecord`: tool name, validated arguments, result summary, artifact IDs, errors, latency.
- `TeachingLogEntry`: teacher-facing event summary with links to memory changes and trace IDs.
- `TraceLogEntry`: developer-facing model/tool event with route, tokens, retries, validation, cost estimate, raw artifact references.
- `LongTaskState`: goal, current status, evidence, pending clarification, completed tool calls, next suggested actions.
- `MemoryPatch`: target wiki pages, proposed changes, confidence, evidence links, pending or stable classification.
- `LearningEvent`: structured reference to a completed interaction that can support memory updates.

## Teaching Skills

Teaching Skills are Markdown behavior policies. They may reference schemas, prompts, tools, and eval fixtures, but tool execution remains centralized in the Tool Runtime.

Foundational skills loaded every turn:

- `teaching-safety`: age-appropriate help, no harmful shortcuts, no confident guessing from ambiguous evidence.
- `student-tone`: warm, direct, student-level explanations, Socratic follow-up when useful.
- `memory-safety`: evidence-linked memory updates, low-confidence observations stay pending.

Task skills loaded on demand:

- `image-ingestion`: when image assets need interpretation.
- `homework-grading`: when a Student Query contains an Assignment Submission.
- `concept-explanation`: when the student asks why or requests a concept explanation.
- `transfer-quiz`: when follow-up practice is useful or requested.

Tool-required skills:

- `write_memory` requires `memory-safety`.
- `grade_attempt` requires `homework-grading`.
- `ingest_image` requires `image-ingestion`.
- `generate_transfer_quiz` requires `transfer-quiz`.

## Tool Runtime Contract

Each Education Tool must provide:

- Tool name.
- Short description for the agent.
- Input schema.
- Output schema.
- Permission level.
- Required Teaching Skills.
- Whether it creates Runtime Artifacts.
- Whether it can write Student Memory Wiki.
- Whether it can call external APIs.
- Error shape.

Initial tools:

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

The first implementation uses Python core tools with this contract. It does not need to expose MCP servers yet, but descriptors should be mappable to MCP-style tools later.

## Permissions

The system is designed for unattended student interaction, not per-message Teacher approval.

Permission levels:

- `read`: read memory, logs, task state, and artifacts.
- `compute`: call models, transform inputs, generate explanations, create temporary artifacts.
- `write_memory`: write Learning Events or update Student Memory Wiki; requires evidence links.
- `external_effect`: download image URLs or call external services; enabled for local dev webhook but logged.
- `destructive`: delete or rewrite durable memory; not available in first-version student flows.

Default Local Live Service behavior:

- Allow `read`, `compute`, `write_memory`, and configured image URL downloads.
- Log all `write_memory` and `external_effect` actions.
- Never allow destructive memory operations from the Teaching Agent.
- Teacher review happens after the fact through Teaching Logs and memory diffs.

## Model Routes

First version routes:

- `vision`: Qwen/VL-compatible multimodal model.
- `agent`: DeepSeek.
- `grader`: DeepSeek.
- `memory`: DeepSeek.
- `summarizer`: DeepSeek.
- `quiz`: DeepSeek.

The runtime should keep route names even though non-vision routes all point to DeepSeek. Route names make traces readable and future model replacement easier.

Each model call records route, model name, latency, retry count, schema validation status, token usage if available, and raw response artifact path.

## Context Assembly

Context is split into a Stable Context Prefix and Dynamic Context Tail.

Stable Context Prefix:

- System contract.
- Foundational Teaching Skills.
- Skill Index.
- Stable Education Tool descriptors.
- Stable output contracts.

Dynamic Context Tail:

- Current Student Query.
- Current Session Summary.
- Long Task State when present.
- Relevant Student Memory Wiki excerpts.
- Recent Teaching Log summary.
- Current image metadata or artifact references.
- Loaded task Teaching Skills.

This ordering is intentional for prompt cache reuse. Dynamic content should be minimized through retrieval and summarization.

## Local Live Service

The first version runs locally, not as a production deployment.

Service responsibilities:

- Expose dev webhook endpoint for Student Query simulation.
- Expose Teacher Dashboard on localhost.
- Submit all incoming queries to the shared Agent Runtime.
- Serve local artifacts needed by the dashboard.
- Keep runtime state in local filesystem directories.

No production authentication, Docker deployment, cloud storage, queue, or Feishu/WeCom online bot is required for the first version.

## Channel Adapters

First-version entrypoints:

- `cli-dev`: submit Student Queries from the terminal for debugging.
- `webhook-dev`: submit Student Queries through local HTTP, including text and image URLs.

Adapter skeletons:

- `feishu`: interface only, no production auth.
- `wecom`: interface only, no production auth.

Adapters convert external messages into `StudentQuery`. They do not grade, update memory, or choose teaching actions.

## Teacher Dashboard

The Teacher Dashboard is read-only and local-only.

Minimum views:

- Student list with recent activity.
- Student detail with profile summary and memory pages.
- Live Session list.
- Teaching Log detail for each turn.
- Memory patch/diff view.
- Low-confidence or pending observations.
- Trace link for developers.

The dashboard should not expose destructive actions in the first version.

## Implementation Phases

### Phase 1: Project Skeleton And Domain Docs

Goal: Create a runnable empty project with the documented boundaries.

- [ ] Create Python package layout under `teach2sq/`.
- [ ] Add `pyproject.toml` with runtime and test dependencies.
- [ ] Add Typer CLI entrypoint with a working `--help` command and no agent behavior yet.
- [ ] Add FastAPI app with a working health route and no agent behavior yet.
- [ ] Add `CONTEXT.md` terms and ADRs already established.
- [ ] Add tests that import the package and verify CLI/app modules load.
- [ ] Commit with message: `chore: scaffold Teach2SQ runtime project`.

### Phase 2: Schemas

Goal: Define the contracts before tool/runtime logic.

- [ ] Implement Pydantic models for Student Query, Image Asset, Agent Turn, Tool Descriptor, Tool Call Record, Teaching Log Entry, Trace Log Entry, Long Task State, Memory Patch, and Learning Event.
- [ ] Export JSON schemas for all public contracts.
- [ ] Add tests for required fields, invalid states, and schema export.
- [ ] Commit with message: `feat: add runtime schema contracts`.

### Phase 3: Artifact And State Store

Goal: Make every runtime action persistable and replayable.

- [ ] Implement artifact ID generation.
- [ ] Implement JSON, JSONL, Markdown, and binary artifact writing.
- [ ] Implement session summary storage.
- [ ] Implement long task state storage.
- [ ] Implement teaching and trace log appenders.
- [ ] Add tests for deterministic paths, append behavior, and corrupt-file handling.
- [ ] Commit with message: `feat: add artifact and state storage`.

### Phase 4: Skill Loader

Goal: Load skills like Claude Code-style behavior policies.

- [ ] Add foundational skill files.
- [ ] Add task skill files.
- [ ] Implement Skill Index generation.
- [ ] Implement stable prefix loading.
- [ ] Implement tool-required skill validation.
- [ ] Add tests for always-loaded skills, agent-selected skills, and required-skill gates.
- [ ] Commit with message: `feat: add teaching skill loader`.

### Phase 5: Tool Runtime

Goal: Register and execute Education Tools through descriptors, not ad hoc function calls.

- [ ] Implement tool registry.
- [ ] Implement permission checks.
- [ ] Implement tool argument validation.
- [ ] Implement tool result validation.
- [ ] Implement tool call trace recording.
- [ ] Stub initial tools with deterministic fake behavior for tests.
- [ ] Add tests for permission denial, schema failures, required skills, and artifact creation.
- [ ] Commit with message: `feat: add education tool runtime`.

### Phase 6: Model Router

Goal: Centralize model calls and trace them.

- [ ] Implement model route config for `vision`, `agent`, `grader`, `memory`, `summarizer`, and `quiz`.
- [ ] Wire `vision` to Qwen/VL-compatible config.
- [ ] Wire non-vision routes to DeepSeek config.
- [ ] Add timeout, retry, and schema-validation hooks.
- [ ] Add recorded-response fixture support.
- [ ] Add tests using fake providers and recorded fixtures.
- [ ] Commit with message: `feat: add model routing`.

### Phase 7: Student Memory Wiki

Goal: Keep durable learning memory readable and evidence-linked.

- [ ] Implement Student Memory Wiki page creation.
- [ ] Implement Learning Event Markdown creation.
- [ ] Implement Memory Patch validation.
- [ ] Implement pending versus stable observation sections.
- [ ] Implement memory search/read helpers.
- [ ] Add tests for evidence links, low-confidence pending entries, and stable updates.
- [ ] Commit with message: `feat: add student memory wiki`.

### Phase 8: Core Education Tools

Goal: Replace stubs with real first-version behavior.

- [ ] Implement `read_student_memory`.
- [ ] Implement `search_student_memory`.
- [ ] Implement `ingest_image_submission` with vision route.
- [ ] Implement `grade_attempt` with DeepSeek route.
- [ ] Implement `explain_concept` with DeepSeek route.
- [ ] Implement `propose_memory_patch` with DeepSeek route.
- [ ] Implement `apply_memory_patch` through the memory layer.
- [ ] Implement `record_learning_event`.
- [ ] Implement `generate_transfer_quiz` with DeepSeek route.
- [ ] Implement `ask_clarification`.
- [ ] Implement `summarize_session`.
- [ ] Implement `finish_response`.
- [ ] Add tests with fake providers and recorded fixtures.
- [ ] Commit with message: `feat: implement first education tools`.

### Phase 9: Agent Runtime Loop

Goal: Let the Teaching Agent decide what to do from Student Query and current context.

- [ ] Implement context assembly with stable prefix and dynamic tail.
- [ ] Implement agent action schema.
- [ ] Implement loop limit and stop conditions.
- [ ] Implement skill selection.
- [ ] Implement tool call execution through Tool Runtime.
- [ ] Implement clarification stop path.
- [ ] Implement final response path.
- [ ] Implement task state recovery.
- [ ] Add tests for direct answer, image grading, clarification, memory update, and transfer quiz flows.
- [ ] Commit with message: `feat: add teaching agent runtime loop`.

### Phase 10: CLI Dev Entrypoint

Goal: Debug the shared runtime without the service.

- [ ] Implement `teach2sq query`.
- [ ] Implement image path input.
- [ ] Implement student/session/task flags.
- [ ] Implement output artifact path reporting.
- [ ] Add CLI tests using fake runtime/model providers.
- [ ] Commit with message: `feat: add CLI dev entrypoint`.

### Phase 11: Local Live Service And Dev Webhook

Goal: Run student interactions through local HTTP.

- [ ] Implement FastAPI app startup.
- [ ] Implement `POST /dev/query`.
- [ ] Implement local image URL download behavior when configured.
- [ ] Submit requests to the shared Agent Runtime.
- [ ] Return final response plus log/artifact IDs.
- [ ] Add service tests with FastAPI test client.
- [ ] Commit with message: `feat: add local live service`.

### Phase 12: Teacher Dashboard

Goal: Let a Teacher inspect activity without reading raw files.

- [ ] Implement student list view.
- [ ] Implement student detail view.
- [ ] Implement session list view.
- [ ] Implement Teaching Log detail view.
- [ ] Implement memory diff/pending observation view.
- [ ] Link Teaching Log entries to Trace Log IDs.
- [ ] Add dashboard tests for rendered pages and local-only route assumptions.
- [ ] Commit with message: `feat: add local teacher dashboard`.

### Phase 13: Channel Adapter Skeletons

Goal: Preserve future Feishu/WeCom integration boundary without implementing production auth.

- [ ] Define Channel Adapter interface.
- [ ] Implement dev webhook adapter.
- [ ] Add Feishu adapter skeleton.
- [ ] Add WeCom adapter skeleton.
- [ ] Add tests that adapters convert sample payloads into Student Query objects.
- [ ] Commit with message: `feat: add channel adapter contracts`.

### Phase 14: Evaluation Harness

Goal: Verify that this is an education harness, not just a chat wrapper.

- [ ] Add example algebra/function assignments.
- [ ] Add example mechanics assignments.
- [ ] Add ambiguous image/text cases.
- [ ] Add expected critical facts for ingestion and grading.
- [ ] Add eval runner using recorded responses.
- [ ] Add manual review checklist for teaching quality.
- [ ] Commit with message: `test: add education harness evals`.

### Phase 15: Documentation And Runbook

Goal: Make the local service usable by another developer.

- [ ] Add README with setup, environment variables, and local run commands.
- [ ] Add guide for creating a Teaching Skill.
- [ ] Add guide for adding an Education Tool.
- [ ] Add guide for reading Teaching Logs and Trace Logs.
- [ ] Add local-only security warning.
- [ ] Commit with message: `docs: add Teach2SQ local runbook`.

## Verification Strategy

Every implementation phase should run:

```bash
uv run pytest
```

When CLI exists:

```bash
uv run teach2sq --help
```

When service exists:

```bash
uv run uvicorn teach2sq.app.server:app --reload
```

When evals exist:

```bash
uv run teach2sq eval examples/assignments
```

The first implementation should prefer fake model providers and recorded fixtures for automated tests. Real Qwen/DeepSeek calls should be manually runnable and trace-recorded, not required for every test run.

## First End-To-End Acceptance Scenario

1. Start Local Live Service.
2. Send a dev webhook Student Query with a math or physics image asset and student text.
3. Agent loads relevant skills and decides whether to ingest image, grade, clarify, update memory, and generate a transfer quiz.
4. Runtime records Trace Log and Teaching Log entries.
5. Student receives a final response or clarification request.
6. Student Memory Wiki receives evidence-linked updates only when appropriate.
7. Teacher Dashboard shows the student, session, Teaching Log, memory update, and trace link.

## Explicit Non-Goals For First Version

- Production Feishu/WeCom deployment.
- Teacher login or access control beyond localhost-only assumptions.
- Docker or cloud deployment.
- Multi-tenant classroom management.
- Destructive memory editing from the agent.
- CAS-level symbolic proof.
- Full curriculum graph.
- Fully autonomous long-running jobs without loop limits and recovery state.

## Risks And Mitigations

**Risk: The open agent loop becomes unpredictable.**
Mitigation: tool descriptors, schemas, permissions, required skills, trace logs, loop limits, and recovery state.

**Risk: Student Memory Wiki accumulates false beliefs.**
Mitigation: evidence links, pending observations, confidence thresholds, Teacher Dashboard review, no destructive writes from student flows.

**Risk: Model costs grow because stable context becomes unstable.**
Mitigation: Stable Context Prefix, compact dynamic tail, recorded fixtures in tests, route-level tracing.

**Risk: Teacher Dashboard drifts into a full admin product.**
Mitigation: first version is read-only and local-only.

**Risk: IM platform details distract from harness design.**
Mitigation: dev webhook first, Feishu/WeCom skeletons only.

## Implementation Defaults

Use these defaults during implementation unless a new ADR explicitly changes them:

- Configure exact DeepSeek and Qwen model names through environment variables, with route names fixed in code.
- Build the Teacher Dashboard as server-rendered FastAPI/Jinja pages first. Add TypeScript only if a specific interaction requires it.
- Write initial Teaching Skill text in the skill Markdown files during Phase 4; prompts belong in skills, not scattered through tools.
- Store generated runtime artifacts under `runtime/artifacts/<date>/<run_id>/`.
- Resolve future hard-to-reverse changes through ADRs only when they are surprising and trade-off driven.
