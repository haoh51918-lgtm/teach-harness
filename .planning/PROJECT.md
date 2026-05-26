# Teach2SQ

## What This Is

Teach2SQ is a local-first agent education harness for middle/high-school math and physics learning. Students send Student Queries that may include homework attempts, image assets, follow-up questions, quiz answers, or requests for explanation; a Teaching Agent runs inside a shared Agent Runtime, selects bounded Education Tools, preserves evidence, updates learning memory conservatively, and returns student-readable help.

The first version is a developer-run Local Live Service with a CLI/debug entrypoint, a dev webhook for simulated live sessions, and a lightweight read-only Teacher Dashboard. It is not a fixed grading pipeline, not a deterministic multi-agent system, and not a production IM bot.

## Core Value

Students receive reliable, age-appropriate math and physics help while every teaching action, model judgment, artifact, and durable memory update remains evidence-linked and auditable by a Teacher.

## Requirements

### Validated

(None yet - ship to validate)

### Active

- [ ] Create a Python/FastAPI/Typer project skeleton with clear Teach2SQ package boundaries.
- [ ] Define Pydantic schema contracts for Student Queries, image assets, assignments, ingested problems, grading results, clarification requests, quiz sets, agent turns, tools, logs, task state, memory patches, and learning events.
- [ ] Implement artifact and state storage for JSON, JSONL, Markdown, binary assets, session summaries, long task state, Teaching Logs, and Trace Logs.
- [ ] Implement Teaching Skill loading with always-loaded foundational skills, on-demand task skills, a Skill Index, stable context prefix assembly, and tool-required skill validation.
- [ ] Implement an Education Tool Runtime with descriptors, schemas, permissions, validation, artifact contracts, and trace logging.
- [ ] Implement model routing with named routes for vision, agent, grader, memory, summarizer, and quiz; use Qwen/VL-compatible vision and DeepSeek-compatible non-vision routes in the first version.
- [ ] Implement a Markdown-first Student Memory Wiki with evidence-linked Learning Events, Memory Patches, pending observations, stable observations, and read/search helpers.
- [ ] Implement first-version Education Tools: read/search memory, ingest image submissions, grade attempts, explain concepts, propose/apply memory patches, record learning events, generate transfer quizzes, ask clarification, summarize sessions, and finish responses.
- [ ] Implement the open Teaching Agent runtime loop where the model chooses actions from current context rather than following a fixed query-type workflow.
- [ ] Provide a CLI dev entrypoint for submitting Student Queries, image paths, student/session/task identifiers, and receiving artifact/log references.
- [ ] Provide a Local Live Service with a dev webhook that submits Student Queries to the shared Agent Runtime and returns final responses plus artifact/log IDs.
- [ ] Provide a local-only, read-only Teacher Dashboard for students, sessions, Teaching Logs, memory pages, memory diffs, low-confidence observations, and trace links.
- [ ] Add Channel Adapter contracts plus dev webhook, Feishu skeleton, and WeCom skeleton boundaries without implementing production platform deployment.
- [ ] Add an evaluation harness with algebra, functions, mechanics, ambiguous image/text cases, expected critical facts, expected memory updates, and Transfer Quiz targets.
- [ ] Add documentation and runbooks for local setup, environment variables, Teaching Skills, Education Tools, logs, traces, and local-only security assumptions.

### Out of Scope

- Production Feishu/WeCom deployment - first version keeps only local dev webhook plus adapter skeletons.
- Teacher login, production authentication, and access control - first version assumes localhost-only access.
- Docker, cloud deployment, queueing, or production storage - first version is a local developer-run service.
- Multi-tenant classroom management - first version focuses on the harness, logs, memory, and local live loop.
- A fixed grading pipeline or deterministic MAS - the Teaching Agent must choose tools through the Agent Runtime.
- Separate autonomous grader, memory, quiz, or ingestion agents - these are Education Tools and Teaching Skills inside one Teaching Agent loop.
- Destructive Student Memory Wiki edits from student flows - durable memory updates must be conservative and evidence-linked.
- CAS-level symbolic proof or a full curriculum graph - the first scope is practical middle/high-school algebra, functions, and mechanics support.
- Per-message Teacher approval - Teachers review Teaching Logs and memory changes after the fact.

## Context

Teach2SQ already has a strong domain language in `CONTEXT.md` and four ADRs:

- `docs/adr/0001-open-agent-loop-with-teaching-skills.md`: use an open Teaching Agent loop constrained by Teaching Skills.
- `docs/adr/0002-python-core-tools-with-mcp-compatible-contract.md`: implement first-version Education Tools as Python core tools with MCP-compatible contracts.
- `docs/adr/0003-shared-agent-runtime-for-cli-and-live-sessions.md`: keep one shared Agent Runtime for CLI and live sessions.
- `docs/adr/0004-preserve-harness-over-deterministic-mas.md`: prevent regression into deterministic MAS architecture.

The current detailed implementation plan is `docs/superpowers/plans/2026-05-24-teach2sq-agent-runtime-plan.md`. It describes the desired local Claude Code-like harness shape: context assembly, skill loading, model routing, tool dispatch, permission gates, artifact recording, memory persistence, session compaction, task recovery, and teacher/developer audit surfaces.

The first educational focus is middle/high-school algebra, functions, and mechanics. Student inputs may include Markdown/JSON text, local image files, and IM-provided image URLs. Vision ingestion should preserve problem text, diagrams, known conditions, goals, LaTeX formulas, units, and solution steps. Grading should support rubric scores when a rubric exists, but otherwise use correctness levels, confidence, error locations, conceptual diagnosis, corrected reasoning, and student-facing Markdown feedback without fake precision.

Student Memory Wiki is not chat history. It stores durable learning observations, error patterns, quiz history, and evidence-linked Learning Events. Low-confidence observations stay pending until confirmed by future evidence. Runtime Artifacts, Trace Logs, Session Summaries, and Long Task State remain operational records rather than memory pages.

## Constraints

- **Architecture**: Preserve a single Teaching Agent loop and shared Agent Runtime - this is the core product shape and is protected by ADR-0004.
- **Tooling**: First-version Education Tools are Python core tools with Pydantic schemas, descriptors, permission metadata, artifact outputs, and auditable call records - this keeps the first build tractable while preserving future MCP/plugin compatibility.
- **Runtime**: CLI, dev webhook, and Local Live Service must all invoke the same Agent Runtime - CLI is a debugging entrypoint, not the architecture.
- **State**: Keep four state layers separate: Session Summary, Long Task State, Student Memory Wiki, and Teaching/Trace Logs - mixing them would make recovery and audit unreliable.
- **Models**: Use named routes even when several routes point to DeepSeek - route names keep traces readable and allow later model replacement.
- **Vision**: Use a Qwen/VL-compatible route for multimodal ingestion - image ambiguity must produce clarification rather than confident grading.
- **Memory Safety**: Memory writes require evidence links and confidence handling - one low-confidence event cannot become a stable long-term trait.
- **Dashboard**: Teacher Dashboard is read-only and local-only in v1 - it should not become a production admin/control plane.
- **Testing**: Automated tests should prefer fake model providers and recorded fixtures - real Qwen/DeepSeek calls are manually runnable and trace-recorded, not mandatory for every test run.
- **Reference Discipline**: When uncertain about harness architecture, agent loop behavior, tool use, skills, context compression, permissions, memory, task state, recovery, or MCP/plugin direction, inspect the referenced Claude Code learning projects before inventing a mechanism.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build Teach2SQ as an Agent Education Harness rather than a grading pipeline | Student Queries vary, long tasks need recovery, and the agent needs open tool choice constrained by skills | - Pending |
| Use one Teaching Agent loop instead of fixed worker agents | Prevents deterministic MAS regression and keeps educational behavior in skills/tools | - Pending |
| Implement first-version tools as Python core tools with MCP-compatible contracts | Faster local build while preserving future external tool exposure | - Pending |
| Share one Agent Runtime across CLI and live sessions | Keeps context assembly, tools, permissions, artifacts, memory, and recovery consistent | - Pending |
| Store Student Memory Wiki as Markdown with evidence-linked patches | Teachers need readable durable memory, and memory must not become raw chat history | - Pending |
| Build Local Live Service and dev webhook before production Feishu/WeCom integration | Keeps focus on harness design and local 24/7 interaction behavior | - Pending |
| Make Teacher Dashboard read-only and local-only in v1 | Supports audit without turning the dashboard into an admin product | - Pending |
| Keep route names for vision, agent, grader, memory, summarizer, and quiz | Makes traces readable and model replacement easier | - Pending |
| Use clarification when image/formula/unit ambiguity can change grading | Avoids teaching or grading from unreliable evidence | - Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `$gsd-transition`):
1. Requirements invalidated? -> Move to Out of Scope with reason
2. Requirements validated? -> Move to Validated with phase reference
3. New requirements emerged? -> Add to Active
4. Decisions to log? -> Add to Key Decisions
5. "What This Is" still accurate? -> Update if drifted

**After each milestone** (via `$gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check - still the right priority?
3. Audit Out of Scope - reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-26 after initialization*
