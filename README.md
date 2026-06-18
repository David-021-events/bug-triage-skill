# bug-triage — Claude Code skill

Turns a raw bug report (pasted from email, WhatsApp, or anywhere) into a structured Linear ticket draft. Always shows you the draft and waits for your approval before writing anything to Linear.

## Requirements

- [Claude Code](https://claude.ai/code)
- The **Linear** MCP connector enabled in Claude Code settings

## Install

```bash
git clone https://github.com/YOUR_USERNAME/bug-triage-skill ~/.claude/skills/bug-triage
```

Then restart Claude Code (or start a new session). The skill is available immediately as `/bug-triage` or `/triage`.

## Usage

Paste a bug report and say "triage this", or just invoke `/triage` directly.

## Setting a default Linear project

The skill will ask which project to file bugs in the first time you use it each session. To skip that prompt, add a line to your project's `CLAUDE.md`:

```
Linear bug project: your-project-name
```

## License

MIT
