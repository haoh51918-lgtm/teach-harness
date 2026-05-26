# Stack Research: Teach2SQ

**Date:** 2026-05-26
**Milestone context:** Greenfield local-first agent education harness

## Recommendation

Use the documented stack from `docs/superpowers/plans/2026-05-24-teach2sq-agent-runtime-plan.md` with small clarifications:

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

FastAPI remains a good fit for a local service that needs typed request/response schemas, dependency injection, test-client friendliness, and later adapter boundaries. Its dependency injection system is useful for passing runtime config, state stores, model routers, permission policy, and fake providers in tests without inventing a plugin framework.

Source: https://fastapi.tiangolo.com/tutorial/dependencies/

### Typer

Typer is still the natural CLI companion because it is built around Python type hints and fits the same style as FastAPI. Use it for `teach2sq query`, `teach2sq eval`, and debug/admin commands that call the shared Agent Runtime rather than duplicating behavior.

Source: https://typer.tiangolo.com/

### Pydantic v2

Pydantic v2 is the right center for contracts. It validates untrusted data, serializes models, and emits JSON Schema. Use strict schemas for tool inputs/outputs and model responses, but do not rely on model-side JSON mode alone. DeepSeek JSON output can enforce valid JSON shape at the API level, but Teach2SQ still needs Pydantic validation, retries, and failed-turn logging.

Sources:
- https://pydantic.dev/docs/validation/2.0/usage/models/
- https://pydantic.dev/docs/validation/2.0/usage/json_schema
- https://api-docs.deepseek.com/news/news0725/

### DeepSeek

DeepSeek currently documents OpenAI/Anthropic-compatible API formats. As of 2026-05-26, its quick-start docs list `deepseek-v4-flash` and `deepseek-v4-pro`; `deepseek-chat` and `deepseek-reasoner` are documented as scheduled for deprecation on 2026-07-24. Teach2SQ should keep route names (`agent`, `grader`, `memory`, `summarizer`, `quiz`) fixed in code but configure exact provider model IDs through environment variables.

Sources:
- https://api-docs.deepseek.com/
- https://api-docs.deepseek.com/news/news0725/

### Qwen/VL

Alibaba Cloud Model Studio's visual understanding docs support single and multiple image inputs and OpenAI-compatible requests for remote image content. Local file paths are supported by DashScope Python/Java SDKs but not by DashScope HTTP or OpenAI-compatible requests, so Teach2SQ should normalize local image assets into provider-supported forms through the vision route. The docs also expose image token accounting, making token/cost estimation a first-version concern for multi-image homework submissions.

Source: https://www.alibabacloud.com/help/en/model-studio/vision/

### pytest

pytest remains the right testing base. Use `tmp_path` for isolated filesystem state, `monkeypatch` for environment/model-provider substitution, and fake/recorded providers to avoid real model calls in normal test runs.

Source: https://pytest.org/en/8.0.x/reference/fixtures.html

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
