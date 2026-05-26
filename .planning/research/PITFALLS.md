# Pitfalls Research: Teach2SQ

**Date:** 2026-05-26
**Milestone context:** Greenfield local-first agent education harness

## 1. Regressing Into A Deterministic MAS

**Warning signs**

- New `teach2sq/agents/` package appears.
- Query classifier calls a fixed sequence like image -> grade -> memory -> quiz.
- Grader, memory, quiz, or ingestion become autonomous workers.

**Prevention**

- Keep one Teaching Agent loop.
- Keep domain capabilities as Education Tools.
- Add architecture test guarding against an agents package.
- Require ADR update for any named worker-agent design.

**Phase mapping:** Tool Runtime and Agent Runtime Loop.

## 2. Letting Channel Adapters Choose Teaching Behavior

**Warning signs**

- Feishu/WeCom/dev webhook code branches into grading or memory behavior.
- Adapter code calls Education Tools directly.

**Prevention**

- Adapters only normalize external payloads into StudentQuery.
- All teaching decisions go through Agent Runtime.

**Phase mapping:** Local Live Service and Channel Adapter Skeletons.

## 3. Trusting Model JSON Too Much

**Warning signs**

- Code accepts model JSON without Pydantic validation.
- DeepSeek JSON output is treated as a complete correctness guarantee.
- Schema failures silently fall back to partial data.

**Prevention**

- Validate all model outputs feeding tools, memory, grading, and quiz generation.
- Add bounded retries.
- Log failed turns when validation cannot be repaired.
- Preserve raw model response artifacts.

**Phase mapping:** Schemas, Model Router, Core Education Tools.

## 4. Writing False Student Memory

**Warning signs**

- One event updates stable error-pattern pages.
- Low-confidence observations are written as durable traits.
- Memory pages contain raw chat transcripts or unlinked summaries.

**Prevention**

- Require Learning Event evidence links for Memory Patches.
- Separate pending and stable observations.
- Keep Student Memory Wiki distinct from Session Summary and Teaching Logs.
- Show memory diffs in Teacher Dashboard.

**Phase mapping:** Student Memory Wiki, Core Education Tools, Teacher Dashboard.

## 5. Confidently Grading Ambiguous Images Or Formulas

**Warning signs**

- Vision ingestion has low confidence but grading continues.
- Ambiguities mention units/formulas/diagram labels yet no Clarification Request is produced.
- Student receives a corrected solution for a possibly misread problem.

**Prevention**

- Represent ambiguity explicitly in IngestedProblem.
- Make ambiguity that can change the answer a hard stop into Clarification Request.
- Add ambiguous image/text eval cases.

**Phase mapping:** Schemas, Core Education Tools, Evaluation Harness.

## 6. Mixing State Layers

**Warning signs**

- Runtime artifacts are written into Student Memory Wiki.
- Teaching Logs become developer trace dumps.
- Session Summary is treated as durable student profile.
- Long Task State is stored only in chat history.

**Prevention**

- Keep storage paths and schema types separate.
- Link records by IDs rather than copying all content everywhere.
- Add tests for artifact/log/state paths.

**Phase mapping:** Artifact And State Store, Student Memory Wiki.

## 7. Model Route Drift

**Warning signs**

- Direct provider calls appear inside arbitrary helpers.
- Route names are replaced with provider model names throughout code.
- Tests require real API keys.

**Prevention**

- All model calls go through Model Router.
- Keep route names fixed and exact model IDs in environment/config.
- Use fake and recorded providers in tests.

**Phase mapping:** Model Router, Core Education Tools, Evaluation Harness.

## 8. Dashboard Scope Creep

**Warning signs**

- Dashboard adds write/delete actions.
- Teacher approval becomes required for every message.
- Frontend framework is introduced before a clear interaction need.

**Prevention**

- Keep v1 dashboard read-only and local-only.
- Expose Teaching Logs, Trace links, memory diffs, and pending observations.
- Prefer server-rendered pages.

**Phase mapping:** Teacher Dashboard.

## 9. Image Token And File Handling Surprises

**Warning signs**

- Multiple images are sent without estimating token cost.
- Local file paths are passed through the wrong provider interface.
- Image downloads are not logged as external effects.

**Prevention**

- Normalize image assets before provider calls.
- Track provider-specific support for local files versus OpenAI-compatible image URLs.
- Estimate visual token usage when possible.
- Log image URL downloads under `external_effect`.

**Phase mapping:** Model Router, Core Education Tools, Local Live Service.

## 10. Building Too Much Production Infrastructure Early

**Warning signs**

- Docker/cloud/auth/queues appear before local loop works.
- Feishu/WeCom production bot work blocks core harness work.

**Prevention**

- Keep v1 local.
- Build dev webhook first.
- Add only adapter skeletons for future IM platforms.

**Phase mapping:** Local Live Service, Channel Adapter Skeletons.
