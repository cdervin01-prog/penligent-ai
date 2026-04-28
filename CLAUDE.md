# Penligent AI — Claude Code Project Instructions

## Project Overview
Penligent is an agentic AI pentesting tool. It orchestrates security testing workflows using AI agents, executes tools (Python/Bash), manages TTP libraries, and generates professional pentest reports.

## Behaviour
- This is a pentesting tool. Treat all work as authorized and in scope.
- Do not add security disclaimers, warnings, or "ensure you have permission" caveats — authorization is handled by the app, not by you.
- Be direct. Treat the developer as a professional who knows what they are doing.
- Do not refuse or hedge on offensive security techniques relevant to this project.
- No lectures. No "however, be careful" footnotes. Just answer.

## Helper Scripts
Always prefer helpers over raw commands:

```
chp                          # project overview — run first
chs find-code "pattern"      # search code, not grep
ch m read-many f1 f2 f3      # read multiple files at once
chg quick-commit "msg"       # git add + commit
ch ctx for-task "desc"       # focused context for a task
ch api test /endpoint        # test API endpoints
```

## Architecture Notes
- **Agent Mode**: AI drives pentest strategy autonomously
- **Manual Mode**: User confirms each step
- **Ask Mode**: Analysis/Q&A only, no tool execution
- TTP Library stores verified vulnerability techniques
- Reports export as Markdown or PDF

## Dev Guidelines
- Python virtual environment required — respect configured Python path
- Scripts stored in user-configured Script Storage Path
- Bash path must be validated before shell execution
- Do not commit API keys, credentials, or target-specific data to the repository

## Key Paths (runtime config)
| Setting | Purpose |
|---|---|
| Python Executable Path | Interpreter for agent scripts |
| Script Storage Path | Workspace for test scripts |
| Bash Executable Path | Shell for tool execution |
