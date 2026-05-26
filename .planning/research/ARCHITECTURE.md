# Architecture Research: Teach2SQ

**Date:** 2026-05-26
**Milestone context:** Greenfield local-first agent education harness

## Recommended Architecture

Teach2SQ should be implemented as one shared Agent Runtime with deterministic infrastructure around an open Teaching Agent loop:

```text
CLI / Dev Webhook / Channel Adapter
        |
        v
StudentQuery
        |
        v
Agent Runtime
  - context builder
  - skill loader
  - model router
  - tool runtime
  - permissions
  - artifact/state store
  - recovery
        |
        +--> Teaching Skills
        +--> Education Tools
        +--> Student Memory Wiki
        +--> Teaching Logs
        +--> Trace Logs
        +--> Runtime Artifacts
        |
        v
Student-facing Markdown response
```

The Teaching Agent chooses actions, but every action is constrained by schemas, tool descriptors, permissions, skill gates, loop limits, validation, and logs.

## Component Boundaries

### `teach2sq/schemas/`

Owns Pydantic contracts. No model calls, no filesystem writes except schema export helpers.

Key schemas:

- StudentQuery
- ImageAsset
- AssignmentSubmission
- IngestedProblem
- SolutionStep
- GradingResult
- ClarificationRequest
- QuizSet
- AgentTurn
- ToolDescriptor
- ToolCallRecord
- TeachingLogEntry
- TraceLogEntry
- LongTaskState
- MemoryPatch
- LearningEvent

### `teach2sq/runtime/`

Owns shared runtime behavior:

- `agent_loop.py`: loop control and action handling.
- `context_builder.py`: Stable Context Prefix and Dynamic Context Tail.
- `skill_loader.py`: foundational skills, task skills, Skill Index, required-skill validation.
- `model_router.py`: named route dispatch, provider config, retries, raw response artifacts.
- `permissions.py`: permission checks and policy decisions.
- `artifacts.py`: artifact IDs and artifact writes.
- `state_store.py`: session summaries, long task state, Teaching Logs, Trace Logs.

Runtime code should not contain domain prompts scattered through helper functions. Teaching behavior belongs in skills and route/tool contracts.

### `teach2sq/tools/`

Owns Education Tool registration and deterministic execution.

- `registry.py`: register and look up tools.
- `descriptors.py`: expose stable tool descriptors.
- `vision_tools.py`
- `grading_tools.py`
- `memory_tools.py`
- `quiz_tools.py`
- `logging_tools.py`

Tools can call Model Router when the tool contract requires model work, but all model calls should be trace-recorded and validated.

### `teach2sq/skills/`

Owns Markdown Teaching Skills. Skills are behavior policies, not Python workers.

Foundational skills are always loaded. Task skills are selected by the agent or required by tools.

### `teach2sq/memory/`

Owns Student Memory Wiki operations:

- page layout
- Learning Event Markdown rendering
- Memory Patch validation and application
- pending/stable observation sections
- read/search helpers

### `teach2sq/app/`

Owns FastAPI app composition:

- `server.py`
- `routes_dev_webhook.py`
- `routes_teacher_dashboard.py`

App routes submit Student Queries to the shared Agent Runtime. They do not choose teaching behavior.

### `teach2sq/cli.py`

Owns Typer CLI commands. CLI is a debug entrypoint into the shared Agent Runtime, not a separate implementation.

## Data Flow

### Homework with image

1. CLI/dev webhook receives text and image assets.
2. Adapter normalizes payload into StudentQuery.
3. Agent Runtime builds context from stable skills/tools plus dynamic query/session/memory/task state.
4. Teaching Agent selects image-ingestion skill and `ingest_image_submission` if image evidence is needed.
5. Tool Runtime validates arguments and permissions.
6. Vision route calls Qwen/VL-compatible provider and writes raw/structured artifacts.
7. IngestedProblem is validated by Pydantic.
8. Teaching Agent chooses `grade_attempt`, `ask_clarification`, `propose_memory_patch`, `generate_transfer_quiz`, or final answer based on evidence.
9. Runtime writes Teaching Log, Trace Log, Runtime Artifacts, Learning Event, and any Memory Patch.
10. Student receives Markdown response or Clarification Request.

### Concept explanation without assignment evidence

1. Student Query enters runtime.
2. Context builder loads relevant memory excerpts and recent session summary.
3. Teaching Agent may answer directly or call `explain_concept`.
4. Runtime logs model/tool actions and final response.
5. No Student Memory Wiki update occurs unless a tool proposes evidence-linked memory from a Learning Event.

## Build Order Implications

1. Skeleton and dependency setup.
2. Schemas.
3. Artifact/state store.
4. Skill files and loader.
5. Tool Runtime.
6. Model Router.
7. Student Memory Wiki.
8. Core Education Tools.
9. Teaching Agent runtime loop.
10. CLI.
11. Local Live Service/dev webhook.
12. Teacher Dashboard.
13. Channel Adapter skeletons.
14. Evaluation harness.
15. Documentation/runbook.

This order matches the dependency graph. Avoid building the dashboard or live service deeply before logs/artifacts/state exist.

## Architecture Checks To Add

- Test that importing `teach2sq` does not require provider credentials.
- Test that CLI and FastAPI both call the same runtime facade/factory.
- Test that all tools have descriptors, schemas, permissions, required skills, and artifact metadata.
- Test that tool results are Pydantic-validated before use.
- Test that `write_memory` is impossible without evidence links.
- Test that no `teach2sq/agents/` package exists unless ADR-0004 is revised.
- Regression test proving the same query type can lead to different tool sequences depending on context.

## References

- FastAPI dependency injection: https://fastapi.tiangolo.com/tutorial/dependencies/
- Typer type-hint CLI model: https://typer.tiangolo.com/
- Pydantic validation and JSON schema: https://pydantic.dev/docs/validation/2.0/usage/models/
- DeepSeek OpenAI/Anthropic-compatible API: https://api-docs.deepseek.com/
- Alibaba Cloud Qwen visual understanding: https://www.alibabacloud.com/help/en/model-studio/vision/
