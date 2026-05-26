# Research Summary: Teach2SQ

**Date:** 2026-05-26

## Stack

Use the planned Python stack: FastAPI for the local service and dashboard routes, Typer for CLI, Pydantic v2 for contracts and validation, pytest for tests, Markdown for Student Memory Wiki, JSON/JSONL for runtime artifacts and traces, DeepSeek-compatible non-vision routes, and Qwen/VL-compatible vision route. Keep exact model IDs in environment/config and preserve route names in code.

## Table Stakes

Teach2SQ v1 needs:

- shared Agent Runtime for CLI, dev webhook, and Local Live Service;
- schemas before runtime/tool behavior;
- artifact/state storage;
- Teaching Skill loader and Skill Index;
- Education Tool Runtime with descriptors, schemas, permission gates, artifact contracts, and trace logging;
- Model Router with fake/recorded providers for tests;
- Student Memory Wiki with evidence-linked patches and pending/stable observations;
- core education tools for memory, image ingestion, grading, explanation, clarification, session summary, learning events, and transfer quizzes;
- open Teaching Agent loop with loop limits and recovery;
- local-only read-only Teacher Dashboard;
- evaluation harness with ambiguous cases.

## Watch Out For

The largest risks are architectural drift and unsafe memory. Do not turn tools into worker agents. Do not let adapters choose teaching behavior. Do not write raw model output or low-confidence claims into memory. Do not grade when image/formula/unit ambiguity can change the answer. Do not let dashboard scope creep into production admin behavior.

## Requirements Implications

Requirements should be written around verifiable contracts and runtime behavior rather than UI screens first. The roadmap should build foundation-first but still preserve end-to-end slices once the runtime core is ready:

1. Skeleton and contracts.
2. Artifact/state/log storage.
3. Skills and tool runtime.
4. Model routing and fake providers.
5. Memory wiki.
6. Core tools.
7. Agent loop.
8. CLI/service.
9. Dashboard/adapters/evals/docs.

## Source Links

- FastAPI dependency injection: https://fastapi.tiangolo.com/tutorial/dependencies/
- Typer CLI docs: https://typer.tiangolo.com/
- Pydantic models and validation: https://pydantic.dev/docs/validation/2.0/usage/models/
- Pydantic JSON schema: https://pydantic.dev/docs/validation/2.0/usage/json_schema
- DeepSeek API quick start: https://api-docs.deepseek.com/
- DeepSeek JSON output/tool-call notes: https://api-docs.deepseek.com/news/news0725/
- Alibaba Cloud Qwen visual understanding: https://www.alibabacloud.com/help/en/model-studio/vision/
- pytest fixtures: https://pytest.org/en/8.0.x/reference/fixtures.html
