# Teach2SQ Agent Education Harness Design

Date: 2026-05-23
Status: Superseded by `docs/superpowers/plans/2026-05-24-teach2sq-agent-runtime-plan.md`

## Purpose

Teach2SQ is a headless agent education harness for math and physics learning. The first version validates one end-to-end learning loop:

1. A student submits a homework attempt as image plus text.
2. A multimodal model converts the submission into structured text, LaTeX formulas, and solution steps.
3. DeepSeek grades the work, explains mistakes, and produces student-facing feedback.
4. The system updates a per-student Markdown wiki with weak knowledge points and repeated error patterns.
5. The system generates targeted transfer questions based on the latest error and the student wiki.

The first version is a CLI-first harness, not a web app. It should be easy to call from a future Feishu or WeCom TypeScript adapter.

## Scope

This spec covers a thin end-to-end slice across the full system. Each subsystem should be useful but narrow.

In scope:

- Initial support for middle/high-school algebra, functions, and mechanics problems.
- Markdown and JSON assignment inputs.
- Local image file paths and IM image URLs as image sources.
- Qwen-style multimodal ingestion for image plus text submissions.
- DeepSeek grading, explanation, memory patching, and transfer-question generation.
- Student-readable Markdown output and machine-readable JSON output.
- Markdown-only student memory wiki and event pages.
- Python-first implementation with stable JSON schemas for future TypeScript adapters.
- CLI commands for both full runs and individual pipeline stages.

Out of scope for the first version:

- Web UI.
- Multi-user account system.
- Teacher dashboard.
- Online Feishu or WeCom bot deployment.
- Full curriculum knowledge graph.
- CAS-level symbolic proof or algebra verification.
- Large-scale automated scoring platform.

## Architecture

The harness is Python-first. It exposes a CLI and passes structured JSON objects between internal stages. TypeScript is kept at the contract boundary: future IM adapters should translate chat messages into the same task schema and call the Python harness.

Core modules:

- `intake`: Read Markdown or JSON homework input, local image paths, and IM image URLs. Normalize them into `AssignmentTask`.
- `vision_ingest`: Call a multimodal model such as Qwen to transcribe images, normalize formulas into LaTeX, and split the student work into semantic steps.
- `grader`: Call DeepSeek to evaluate the structured problem, locate errors, explain corrections, and decide whether clarification is required.
- `memory`: Maintain a per-student Markdown wiki with knowledge pages, error-pattern pages, event pages, and quiz pages.
- `quiz`: Generate one to three transfer questions from the latest error and relevant student memory.
- `cli`: Provide `run` for the complete pipeline and stage commands for inspection and evaluation.

The first version does not need a service process. A future TypeScript adapter can call the CLI first; a local HTTP wrapper can be added only if IM deployment needs long-running process behavior.

## Data Flow

The complete pipeline is:

```text
AssignmentTask
  -> IngestedProblem
  -> GradingResult
  -> MemoryPatch
  -> QuizSet
  -> RunReport
```

`AssignmentTask` is the unified input. It includes student ID, subject hints, grade-level hints, problem text, student notes, optional reference answer, optional rubric, and image assets. Image assets may be local files or IM image URLs.

`IngestedProblem` is the multimodal ingestion result. It includes the transcribed problem, image description, known conditions, target quantity or proof goal, LaTeX formulas, student solution steps, unit and dimension information, confidence scores, and ambiguities.

`GradingResult` is the grading result. If a rubric is available, it may include section scores and a total score. If no rubric is available, it should include correctness level, confidence, error locations, conceptual misunderstandings, corrected reasoning, and student-facing Markdown feedback.

`MemoryPatch` is the proposed student wiki update. It records weak knowledge points, repeated error patterns, mastery changes, evidence links, and low-confidence observations that must remain pending until validated by future events.

`QuizSet` contains one to three transfer questions. Each question includes the prompt, target knowledge point, expected solution, answer explanation, and the reason it matches the student's current error pattern.

`RunReport` is the final output bundle. It contains student-facing Markdown and machine-readable JSON for evaluation, replay, IM adapters, and debugging.

## Schema Boundaries

The implementation should define JSON schemas for the major pipeline objects:

- `AssignmentTask`
- `ImageAsset`
- `IngestedProblem`
- `SolutionStep`
- `GradingResult`
- `ClarificationRequest`
- `MemoryPatch`
- `QuizSet`
- `RunReport`

Every model-facing stage must validate model output against the relevant schema. Schema validation failures should trigger bounded retries and then produce a clear failed run report.

## CLI Design

The CLI should support a full end-to-end command:

```bash
teach2sq run assignment.md --student s001 --image ./work.jpg
```

It should also support stage commands:

```bash
teach2sq ingest assignment.md --image ./work.jpg
teach2sq grade ingested-problem.json --student s001
teach2sq remember grading-result.json --student s001
teach2sq quiz --student s001 --from latest
```

The `run` command should produce a complete report. Stage commands should help debug whether a problem came from image transcription, formula normalization, grading, memory update, or quiz generation.

All commands should be able to write JSON and Markdown artifacts to disk for replay and evaluation.

## Student Memory Wiki

The first version stores student memory entirely as Markdown files. This keeps memory readable, auditable, and easy to edit by hand.

Suggested layout:

```text
students/
  <student_id>/
    profile.md
    knowledge/
      algebra-functions.md
      mechanics.md
    error-patterns/
      sign-errors.md
      unit-dimension-errors.md
      force-analysis-errors.md
    events/
      2026-05-23-001-assignment.md
    quizzes/
      2026-05-23-001-transfer-quiz.md
```

`profile.md` summarizes the student's current level, recent weak points, stable error patterns, explanation preferences, and pending observations.

`knowledge/*.md` tracks mastery by topic, such as function graphs, equation transformations, Newton's second law, and force analysis.

`error-patterns/*.md` tracks reusable error modes, such as sign mistakes, missing units, treating vectors as scalars, or incorrect force directions.

`events/*.md` stores each homework event, including input summary, Qwen transcription, DeepSeek grading, confidence, and evidence links.

`quizzes/*.md` stores generated transfer questions. If the student attempts a follow-up quiz, the same page records the student's answer and outcome.

Memory follows a wiki-style compilation model. Raw events are preserved as evidence pages. Longer-lived memory pages are updated from event evidence rather than from unstructured chat history.

High-confidence observations may update knowledge and error-pattern pages directly. Low-confidence observations must go into pending sections and should not become stable profile facts until repeated or confirmed.

## Model Reliability

Reliability has three layers.

First, schema validation is mandatory for model outputs. Missing required fields, invalid field types, empty solution steps, or malformed confidence values should fail validation.

Second, ambiguity handling must be explicit. If Qwen cannot clearly read the problem, formula, units, diagram, or student step, it must record the ambiguity. DeepSeek must decide whether the ambiguity affects grading. If it does, the run should produce a clarification request instead of a forced grade.

Third, memory updates must be conservative. A single low-confidence event cannot become a stable long-term student trait. Stable error patterns should emerge from repeated evidence or high-confidence grading.

The system should prefer saying "I need clarification" over confidently teaching from a misread image.

## Evaluation Plan

The first eval set should contain a small number of realistic samples:

- Algebra/function problems with common step errors.
- Mechanics problems involving diagrams, forces, units, or dimensions.
- Cases with readable images.
- Cases with intentionally ambiguous or incomplete images.

Each sample should include expected critical facts:

- Key formulas that must be preserved during ingestion.
- The main student error that should be detected.
- Whether clarification should be requested.
- Expected memory page or error-pattern update.
- Whether the generated transfer question targets the same misconception.

Testing should include:

- Unit tests for schema validation and Markdown artifact writing.
- Golden sample tests for fixed structured outputs.
- Manual education-quality review for explanations, memory patches, and quiz relevance.
- Recorded-response fixtures so real model calls can be replayed during development.

## TypeScript Adapter Contract

The first version does not implement Feishu or WeCom bots. It should still preserve a clean adapter boundary.

A future TypeScript adapter is responsible for:

- Receiving IM messages and image attachments.
- Downloading or referencing image URLs.
- Translating chat input into `AssignmentTask`.
- Calling the Python harness.
- Sending student-facing Markdown feedback back to IM.

The TypeScript layer should not own grading logic, memory logic, or quiz logic. It is a messaging and orchestration layer.

## Implementation Defaults

The first implementation plan should use these defaults unless there is a strong local reason to change them:

- Use a Python package with separate modules for `intake`, `vision_ingest`, `grader`, `memory`, `quiz`, `schemas`, and `cli`.
- Use Typer or an equivalent Python CLI framework with subcommands for `run`, `ingest`, `grade`, `remember`, and `quiz`.
- Use Pydantic models as the source of truth for runtime validation and JSON schema generation.
- Let DeepSeek propose `MemoryPatch`; let the Python memory layer validate, normalize, and apply it to Markdown pages.
- Store run artifacts under an `artifacts/` directory so real model responses can be recorded and replayed.
- Load API keys and model configuration from environment variables and an optional local config file that is not committed.
- During CLI usage, download IM image URLs during `intake` and store a local artifact copy before passing the image to the multimodal provider.

## Success Criteria

The first version is successful when a developer can run one command on a math or physics homework submission and receive:

- A structured transcription of the image and text.
- A validated grading result or a clear clarification request.
- Student-facing Markdown feedback.
- A Markdown event page.
- Updated student wiki pages with evidence links.
- One to three relevant transfer questions.
- Machine-readable JSON artifacts for replay, eval, and future IM adapter use.
