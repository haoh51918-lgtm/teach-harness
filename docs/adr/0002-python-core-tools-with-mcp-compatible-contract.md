# Use Python core tools with an MCP-compatible contract

Teach2SQ will implement first-version Education Tools as Python core tools, not as external MCP servers or plugins. Each tool still needs a descriptor, Pydantic input and output schemas, permission metadata, artifact outputs, and auditable call records so the same contract can later be exposed through MCP or plugin adapters. This keeps the first harness build tractable while preserving the Claude Code-style separation between agent reasoning, tool dispatch, permission gates, and deterministic execution.

