# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-06-18

### Added
- Grounding rules: every triage claim is now evidence-grounded (a report quote or a `file:line`) or explicitly marked as inference.
- Never-fabricate convention: missing repro steps, environment details, or error text are surfaced as `UNKNOWN` / `(?)` gaps, not guessed.
- Confidence labels (`high` / `medium` / `low`) on the cause hypothesis, severity, and priority, with a `⚠ verify before creating` flag on low-confidence items.
- Pre-approval self-check: before asking for approval, the skill checks its own draft (claims sourced? gaps marked? severity/priority justified? duplicate search run?) and surfaces the result.
- Behavioral-classification note: the skill is documented as a Guided Decision (ASK, THEN EXECUTE) workflow.
- Required-tools section with fail-loud behavior when the Linear connector is unavailable (never silently skip the write or report a ticket as created when it wasn't).
- Anti-patterns section listing common triage mistakes to avoid.
- Worked example: new `references/walkthrough.md` demonstrating a full triage flow end-to-end.

## [1.0.0] - Initial release

### Added
- Draft-then-approve workflow: triage a raw bug report into structured fields without writing to Linear until the user explicitly approves.
- Structured draft: title, summary, steps to reproduce, expected vs actual, severity, priority, code-area cause hypothesis, and a gaps list.
- Codebase research to ground the cause hypothesis.
- Duplicate detection that surfaces a candidate and lets the user decide.
- Runtime Linear discovery of teams, projects, statuses, and labels (no hard-coding), with priority mapped to Linear's native scale.
