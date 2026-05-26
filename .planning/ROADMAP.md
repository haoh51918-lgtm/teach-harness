# Roadmap: Teach2SQ

## Overview

Teach2SQ v1 builds an infrastructure-heavy education harness from the inside out: first the project skeleton and contracts, then durable state/logging, skills and tool runtime, model routing and memory, core education tools, the open Teaching Agent loop, local entrypoints, and finally dashboard/evals/docs. The roadmap intentionally preserves the harness architecture: one Teaching Agent loop chooses bounded Education Tools through the shared Agent Runtime.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

- [ ] **Phase 1: Foundation And Contracts** - Create a runnable empty project and the public schema surface.
- [ ] **Phase 2: Artifacts, State, And Logs** - Make runtime activity persistable, replayable, and auditable.
- [ ] **Phase 3: Skills And Tool Runtime** - Load Teaching Skills and execute Education Tools through descriptors and permissions.
- [ ] **Phase 4: Model Routing And Memory Wiki** - Centralize model calls and establish evidence-linked student memory.
- [ ] **Phase 5: Core Education Tools** - Replace stubs with first-version teaching, grading, memory, quiz, and clarification tools.
- [ ] **Phase 6: Agent Loop And CLI** - Let the Teaching Agent choose actions through the runtime and expose CLI debugging.
- [ ] **Phase 7: Local Service And Teacher Dashboard** - Run live local interactions and inspect logs/memory through a read-only dashboard.
- [ ] **Phase 8: Evaluation And Runbook** - Verify educational behavior and document local operation.

## Phase Details

### Phase 1: Foundation And Contracts
**Goal**: Create a runnable Python project with package boundaries, CLI/app stubs, and all public schema contracts.
**Depends on**: Nothing (first phase)
**Requirements**: [FOUND-01, FOUND-02, FOUND-03, FOUND-04, SCHEMA-01, SCHEMA-02, SCHEMA-03, SCHEMA-04, SCHEMA-05, SCHEMA-06]
**Success Criteria** (what must be TRUE):
  1. Developer can import `teach2sq`, run CLI help, and call the FastAPI health route without provider credentials.
  2. All public Pydantic contracts validate representative valid and invalid data.
  3. JSON Schema export exists for all public contracts.
  4. Package boundaries match Teach2SQ language and avoid an `agents` package.
**Plans**: 3 plans

Plans:
- [ ] 01-01: Scaffold Python package, dependencies, CLI stub, FastAPI health route, and tests.
- [ ] 01-02: Implement Student Query, image, assignment, ingestion, grading, clarification, and quiz schemas.
- [ ] 01-03: Implement runtime/tool/log/task/memory schemas plus JSON Schema export and invalid-state tests.

### Phase 2: Artifacts, State, And Logs
**Goal**: Persist runtime artifacts, session summaries, long task state, Teaching Logs, and Trace Logs.
**Depends on**: Phase 1
**Requirements**: [STATE-01, STATE-02, STATE-03, STATE-04, STATE-05]
**Success Criteria** (what must be TRUE):
  1. Runtime can write and reload JSON, JSONL, Markdown, and binary artifacts under deterministic paths.
  2. Teaching Logs and Trace Logs append entries linked to turn IDs and artifact IDs.
  3. Session Summary and Long Task State survive process restarts.
  4. Corrupt or invalid persisted state produces explicit errors.
**Plans**: 2 plans

Plans:
- [ ] 02-01: Implement artifact writer and deterministic path/ID generation.
- [ ] 02-02: Implement session summary, long task state, Teaching Log, and Trace Log stores with tests.

### Phase 3: Skills And Tool Runtime
**Goal**: Load Teaching Skills and execute Education Tools through descriptors, validation, permissions, and trace recording.
**Depends on**: Phase 2
**Requirements**: [SKILL-01, SKILL-02, SKILL-03, SKILL-04, SKILL-05, TOOL-01, TOOL-02, TOOL-03, TOOL-04, TOOL-05]
**Success Criteria** (what must be TRUE):
  1. Foundational skills are always present in the Stable Context Prefix.
  2. Task skills are discoverable through a Skill Index and loadable on demand.
  3. Tool calls are rejected when schemas, permissions, or required skills fail.
  4. Every successful, failed, denied, or validation-error tool call is trace-recorded.
  5. An architecture test prevents accidental deterministic MAS package drift.
**Plans**: 3 plans

Plans:
- [ ] 03-01: Add Teaching Skill files, Skill Index generation, and stable prefix loading.
- [ ] 03-02: Implement tool registry, descriptors, argument/result validation, and required-skill gates.
- [ ] 03-03: Implement permission checks, tool call tracing, deterministic stubs, and architecture guard tests.

### Phase 4: Model Routing And Memory Wiki
**Goal**: Centralize provider calls and establish the Markdown-first evidence-linked Student Memory Wiki.
**Depends on**: Phase 3
**Requirements**: [MODEL-01, MODEL-02, MODEL-03, MODEL-04, MODEL-05, MODEL-06, MEM-01, MEM-02, MEM-03, MEM-04, MEM-05, MEM-06]
**Success Criteria** (what must be TRUE):
  1. All model calls go through named routes with provider config outside tool helpers.
  2. Fake and recorded providers run automated tests without real API keys.
  3. Model calls write traceable raw response artifacts and validation metadata.
  4. Student Memory Wiki pages, Learning Events, and Memory Patches are readable Markdown and schema-validated.
  5. Low-confidence or evidence-free memory updates cannot become stable observations.
**Plans**: 4 plans

Plans:
- [ ] 04-01: Implement route config, fake/recorded providers, and model call tracing.
- [ ] 04-02: Implement retry/schema-validation hooks and Qwen/VL image normalization.
- [ ] 04-03: Implement Student Memory Wiki layout, page creation, and read/search helpers.
- [ ] 04-04: Implement Learning Event rendering, Memory Patch validation, pending/stable safeguards, and tests.

### Phase 5: Core Education Tools
**Goal**: Implement first-version Education Tools for memory, image ingestion, grading, explanation, learning events, quizzes, clarification, summaries, and final responses.
**Depends on**: Phase 4
**Requirements**: [EDU-01, EDU-02, EDU-03, EDU-04, EDU-05, EDU-06, EDU-07, EDU-08, EDU-09]
**Success Criteria** (what must be TRUE):
  1. Teaching Agent can read/search memory, ingest image submissions, grade attempts, ask clarification, and explain concepts through Tool Runtime.
  2. Learning Events and Memory Patches are evidence-linked and memory-safety gated.
  3. Transfer Quiz generation targets recent error patterns.
  4. Session summary and final response tools produce student-readable Markdown and machine-readable artifacts.
**Plans**: 3 plans

Plans:
- [ ] 05-01: Implement memory, learning event, and memory patch tools.
- [ ] 05-02: Implement image ingestion, grading, clarification, and concept explanation tools.
- [ ] 05-03: Implement transfer quiz, session summary, and final response tools with fake/recorded-provider tests.

### Phase 6: Agent Loop And CLI
**Goal**: Implement the open Teaching Agent runtime loop and a CLI dev entrypoint that submits Student Queries to it.
**Depends on**: Phase 5
**Requirements**: [AGENT-01, AGENT-02, AGENT-03, AGENT-04, AGENT-05, AGENT-06, ENTRY-01, ENTRY-02]
**Success Criteria** (what must be TRUE):
  1. Runtime assembles stable and dynamic context for each Student Query.
  2. Teaching Agent can choose direct answer, skill loading, tool call, clarification, state update, or finish.
  3. Runtime executes tool calls only through Tool Runtime and stops safely on all stop conditions.
  4. Long Task State recovery works across turns.
  5. CLI submits text/images and reports final Markdown plus artifact/log references.
**Plans**: 3 plans

Plans:
- [ ] 06-01: Implement context assembly, action schema, loop limits, stop conditions, and skill selection.
- [ ] 06-02: Implement tool execution integration, clarification/final-response paths, task recovery, and open-loop regression tests.
- [ ] 06-03: Implement `teach2sq query` CLI with image path, student/session/task flags, and artifact/log reporting.

### Phase 7: Local Service And Teacher Dashboard
**Goal**: Run local live interactions through FastAPI and expose read-only teacher inspection surfaces.
**Depends on**: Phase 6
**Requirements**: [ENTRY-03, ENTRY-04, ENTRY-05, ENTRY-06, DASH-01, DASH-02, DASH-03, DASH-04, DASH-05, DASH-06]
**Success Criteria** (what must be TRUE):
  1. `POST /dev/query` submits Student Queries to the shared Agent Runtime and returns response/log/artifact IDs.
  2. Configured image URL downloads are logged as external effects.
  3. Feishu and WeCom skeleton adapters convert sample payloads into Student Query objects without production auth.
  4. Teacher Dashboard shows students, sessions, memory pages, Teaching Logs, memory diffs, pending observations, trace links, and no destructive actions.
**Plans**: 3 plans

Plans:
- [ ] 07-01: Implement FastAPI startup, dev webhook, shared runtime wiring, and service tests.
- [ ] 07-02: Implement local image URL download behavior and Channel Adapter contracts/skeletons.
- [ ] 07-03: Implement server-rendered read-only Teacher Dashboard and route tests.

### Phase 8: Evaluation And Runbook
**Goal**: Verify Teach2SQ as an education harness and make local operation understandable to another developer.
**Depends on**: Phase 7
**Requirements**: [EVAL-01, EVAL-02, EVAL-03, EVAL-04, DOC-01, DOC-02]
**Success Criteria** (what must be TRUE):
  1. Developer can run evals for algebra, functions, mechanics, ambiguous cases, memory updates, and Transfer Quiz targets using fake/recorded providers.
  2. Ambiguous evidence evals require clarification instead of confident grading.
  3. README explains setup, environment variables, local commands, and local-only security assumptions.
  4. Docs explain adding Teaching Skills, adding Education Tools, and reading Teaching/Trace Logs.
**Plans**: 2 plans

Plans:
- [ ] 08-01: Implement education harness eval fixtures, runner, and manual review checklist.
- [ ] 08-02: Write README and runbooks for setup, skills, tools, logs, traces, and security assumptions.

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation And Contracts | 0/3 | Not started | - |
| 2. Artifacts, State, And Logs | 0/2 | Not started | - |
| 3. Skills And Tool Runtime | 0/3 | Not started | - |
| 4. Model Routing And Memory Wiki | 0/4 | Not started | - |
| 5. Core Education Tools | 0/3 | Not started | - |
| 6. Agent Loop And CLI | 0/3 | Not started | - |
| 7. Local Service And Teacher Dashboard | 0/3 | Not started | - |
| 8. Evaluation And Runbook | 0/2 | Not started | - |
