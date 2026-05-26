<!-- GSD:project-start source:PROJECT.md -->
## Project

**Teach2SQ**

Teach2SQ is a local-first agent education harness for middle/high-school math and physics learning. Students send Student Queries that may include homework attempts, image assets, follow-up questions, quiz answers, or requests for explanation; a Teaching Agent runs inside a shared Agent Runtime, selects bounded Education Tools, preserves evidence, updates learning memory conservatively, and returns student-readable help.

The first version is a developer-run Local Live Service with a CLI/debug entrypoint, a dev webhook for simulated live sessions, and a lightweight read-only Teacher Dashboard. It is not a fixed grading pipeline, not a deterministic multi-agent system, and not a production IM bot.

**Core Value:** Students receive reliable, age-appropriate math and physics help while every teaching action, model judgment, artifact, and durable memory update remains evidence-linked and auditable by a Teacher.

### Constraints

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
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommendation
- Python 3.12+ as the core runtime.
- FastAPI for the Local Live Service, dev webhook, health route, dashboard routes, and static/artifact serving.
- Typer for CLI dev entrypoints.
- Pydantic v2 for all public contracts, tool inputs/outputs, model-output validation, JSON schema export, and artifact schemas.
- pytest for unit, integration, service, and fixture-based regression tests.
- Jinja2/server-rendered HTML for the first Teacher Dashboard; add TypeScript only when a concrete interactive need appears.
- Markdown files for Student Memory Wiki and teacher-facing logs.
- JSON/JSONL for runtime artifacts, trace logs, model responses, and replayable records.
- DeepSeek-compatible OpenAI/Anthropic API route for non-vision model calls.
- Qwen/VL-compatible Alibaba Cloud Model Studio route for visual understanding.
- Optional OpenAI-compatible client abstraction for providers that expose compatible chat completions.
## Specific Findings
### FastAPI
### Typer
### Pydantic v2
- https://pydantic.dev/docs/validation/2.0/usage/models/
- https://pydantic.dev/docs/validation/2.0/usage/json_schema
- https://api-docs.deepseek.com/news/news0725/
### DeepSeek
- https://api-docs.deepseek.com/
- https://api-docs.deepseek.com/news/news0725/
### Qwen/VL
### pytest
## What Not To Use First
- Do not start with Celery/RQ, cloud queues, Docker, or production deployment scaffolding. The v1 service is local.
- Do not introduce a database until filesystem artifacts and Markdown memory prove insufficient.
- Do not introduce a frontend framework for the Teacher Dashboard before server-rendered pages fail a real workflow.
- Do not expose MCP servers in v1. Keep the core tool contract MCP-compatible.
- Do not make separate worker-agent packages for grading, memory, quiz, or ingestion.
## Confidence
- Python/FastAPI/Typer/Pydantic/pytest: High
- Markdown + JSON/JSONL local storage: High for v1
- DeepSeek route names with env-configured exact models: High
- Qwen/VL as the visual route: Medium-high, with provider-specific request normalization required
- Server-rendered Teacher Dashboard: Medium-high for v1
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
