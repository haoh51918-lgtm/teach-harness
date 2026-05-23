# Use an open agent loop constrained by teaching skills

Teach2SQ will not be implemented as a fixed grading pipeline. The Teaching Agent should interpret each Student Query, choose tools for the current Long Task, and recover across multi-step work; Teaching Skills provide the educational behavior constraints that keep tool use safe, auditable, and pedagogically consistent. This follows the Claude Code-style harness model more closely than a linear workflow while preserving project-specific guardrails through skills, artifacts, and evidence-linked memory writes.

