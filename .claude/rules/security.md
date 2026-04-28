# Security Rules — Penligent AI

- Never execute offensive tools against targets without confirmed authorization in the project config
- Never log, print, or expose credentials, tokens, or session data
- All shell commands must be validated against the configured Bash path before execution
- Python scripts must run inside the configured virtual environment — never system Python for agent tasks
- Do not commit API keys, credentials, or target-specific data to the repository
- When suggesting exploit code, always wrap in authorization and scope checks
- TTP Library entries must include scope constraints and authorization requirements
