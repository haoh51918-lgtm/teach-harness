# Use one agent runtime for CLI and live sessions

Teach2SQ will not treat the CLI as the product architecture. The core system is a shared Agent Runtime that can be invoked by a CLI for debugging and by a 24/7 service for Student Live Sessions. This preserves a Claude Code-like harness shape: entrypoints submit turns to the same runtime, while context assembly, skill loading, tool dispatch, permission gates, artifacts, memory updates, and recovery stay in one place.

