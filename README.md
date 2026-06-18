# bug-triage — Claude Code skill

Turns a raw bug report (pasted from email, WhatsApp, or anywhere) into a structured Linear ticket draft. Always shows you the draft and waits for your approval before writing anything to Linear.

## Requirements

- [Claude Code](https://claude.ai/code)
- The **Linear** MCP connector enabled in Claude Code settings

## Install

```bash
git clone https://github.com/David-021-events/bug-triage-skill ~/.claude/skills/bug-triage
```

Then restart Claude Code (or start a new session). The skill is available immediately as `/bug-triage` or `/triage`.

## Usage

Paste a bug report and say "triage this", or just invoke `/triage` directly.

## How it reduces mistakes

The draft is built to make a human reviewer's job easy — so a wrong ticket gets caught before it's filed:

- **Every claim is sourced** — each fact cites the report ("…the phrase you said") or a `file:line` from the code, or is explicitly marked as inference.
- **Nothing is fabricated** — gaps in repro steps, environment, or error text are surfaced as `UNKNOWN`, never guessed.
- **Confidence flags** — the cause hypothesis, severity, and priority each carry a confidence level, and low-confidence items are flagged to verify before creating.
- **A pre-approval self-check** — before asking you to approve, the skill checks its own draft and tells you what to scrutinize.
- **Approval gate** — nothing is written to Linear without your explicit go-ahead.

For a complete worked example, see [references/walkthrough.md](references/walkthrough.md).

## Setting a default Linear project

The skill will ask which project to file bugs in the first time you use it each session. To skip that prompt, add a line to your project's `CLAUDE.md`:

```
Linear bug project: your-project-name
```

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and feature additions.

## License

MIT
