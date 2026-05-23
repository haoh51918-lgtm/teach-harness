# Preserve harness architecture over deterministic MAS

Teach2SQ must not regress into a deterministic multi-agent workflow with fixed grader, memory, quiz, and ingestion agents coordinated by hard-coded routing. The architecture follows the Claude Code-style harness pattern: a Teaching Agent reasons in a loop, the Agent Runtime assembles context and loads skills, and the Tool Runtime executes bounded Education Tools with schemas, permissions, artifacts, and logs. Deterministic code belongs in the harness infrastructure, not in a fixed MAS control graph that replaces model-driven tool choice.

