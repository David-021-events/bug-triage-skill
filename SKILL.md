---
name: bug-triage
version: "1.1.0"
description: >
  Triage a raw bug report (pasted from email, WhatsApp, or anywhere) into a
  structured draft and, ONLY after the user approves, create a Linear issue.
  Trigger when the user pastes a bug report, forwards a tester complaint, or
  says "triage this", "log this bug", or "turn this into a ticket". Produces a
  draft with title, repro steps, severity, priority, a code-area hypothesis,
  and a list of what's missing — then waits for explicit confirmation before
  writing anything to Linear.
repository: https://github.com/David-021-events/bug-triage-skill
author: David-021-events
license: MIT
tags:
  - bug-triage
  - linear
  - issue-tracking
  - qa
  - workflow
triggers:
  keywords:
    - triage this
    - log this bug
    - turn this into a ticket
    - bug report
    - tester reported
    - user reported
  explicit:
    - /bug-triage
    - /triage
---

# Bug Triage

## Purpose

Turn messy human bug reports into clean, consistent Linear tickets — without
ever creating a ticket the user hasn't seen and approved.

The reports arrive as free-form text (email or WhatsApp) that the user pastes
into the chat. The user is the single human filter: testers report to the user,
the user forwards to this skill.

## Behavioral classification

This is a **Guided Decision** skill: **ASK, THEN EXECUTE.**

The skill drafts a bug report and proposes it to you. It waits for an explicit
"yes, create it" before writing anything to Linear. It will not infer approval
from silence, enthusiasm, or a partial edit. If you change something, it shows
you the full updated draft and asks again.

## The one hard rule

**Never create, update, or modify anything in Linear until the user explicitly
says to.** The whole point of this skill is the confirm gate. Draft first. Show
the draft. Wait for a clear "yes" / "go" / "create it". Only then write.

If the user's message is ambiguous about whether they're approving, ask. Do not
treat "looks good" about one field as approval to create the whole ticket —
approval to create means approval to create the whole thing.

## What "verify" means here (and its limit)

This skill cannot actually reproduce a bug. What it *can*
do is read the repo to form a hypothesis about the likely cause and lay out
steps a human would follow to confirm it. Present reproduction as "steps to
reproduce" for a human to run, never as "I confirmed this."

## Grounding rules

The reviewer can only catch a bad ticket if the draft tells them where to look. So make the draft self-disclosing: show your sources, and never paper over a gap with a guess.

- **Evidence per claim.** Every factual statement in the draft carries its source inline — either a short quoted phrase from the report (e.g. "freezes on checkout") or a `path/to/file.ext:line` from your codebase research. A statement with neither source is inference; mark it as such.
- **Observed vs inferred.** Observed = pulled from the report or read from the code. Inferred = your own analysis joining those dots. Make the difference visible at a glance so the reviewer scrutinizes the inferred parts — that's where errors hide.
- **Never fabricate.** Do not invent version numbers, OS / browser / device, timestamps, error text, stack traces, affected-user counts, or repro details the report didn't supply. If a fact isn't in the report and can't be grounded in the code, write `UNKNOWN` and add it to MISSING / TO CONFIRM — never a plausible-sounding stand-in.
- **Two markers, two jobs.** Use `(?)` inline for a single uncertain token inside a step; use `UNKNOWN` for a whole field you have no grounded value for. Don't blur them.
- **When the report contradicts the code, surface both** — quote the report phrase and cite the `file:line`, and say which one you trust and why. Don't silently pick one.

## Required tools (and what to do if they're missing)

**Linear MCP connector** — required for duplicate search (step 4) and issue creation (step 7). If the connector is not available when the skill reaches either of those steps, it stops and tells you plainly:

> "The Linear MCP connector is not available. I can show you the draft, but I cannot search for duplicates or create the issue. Connect the Linear MCP server and try again."

It never silently skips the create step and never reports a ticket as created when it wasn't. If you want to file the issue manually, the full draft is ready to copy.

**File and grep read access** — used in step 2 to look for a cause hypothesis. If the repo is not readable, the skill proceeds, but the LIKELY CAUSE field is marked `(ungrounded — repo not searched)` and that uncertainty carries through into the MISSING / TO CONFIRM list. The draft is still useful; just treat the cause as speculation with no file evidence behind it.

## Workflow

### 1. Read the report and draft immediately

Do not interrogate the user first. Produce the draft right away from whatever
was pasted in, and flag gaps rather than blocking on them.

### 2. Research the codebase before writing the draft

Before presenting the draft, use file and grep tools to read the code areas
most likely involved. Look in `lib/`, `app/`, `components/`, or wherever the
stack keeps feature code — follow the symptom to the likely file(s). Base the
cause hypothesis on what you actually read. If the repo isn't accessible, say
the guess is ungrounded and offer to refine it once you can see the code.

### 3. Produce the draft in this exact shape

Tag each fact: cite a `"report phrase"` or `file.ext:line` inline, or mark it inference. SUMMARY / EXPECTED / ACTUAL are **OBSERVED** (from the report). LIKELY CAUSE is **INFERRED** (your analysis). Use `(?)` for a single uncertain token; use `UNKNOWN` for a whole field you can't ground — and add it to MISSING / TO CONFIRM. Any `low`-confidence item gets a `⚠ verify before creating` flag so the reviewer knows exactly what to check.

```text
TITLE: <a short, specific, searchable title — describe the symptom, not the guess>

SUMMARY: <1-2 plain sentences on what the tester observed — cite the "report phrase(s)" it's built from>
  [OBSERVED — from the report]

STEPS TO REPRODUCE (for a human to run, not confirmed by me):
  1. <step — cite "report phrase"; mark a guessed token with (?)>
  2. <...>   ← any whole unknown step is UNKNOWN, listed in MISSING / TO CONFIRM
  [OBSERVED where cited; (?) / UNKNOWN where not]

EXPECTED vs ACTUAL:
  Expected: <cite "report phrase">     Actual: <cite "report phrase">
  [OBSERVED — from the report]

SEVERITY: <Critical | High | Medium | Low>  (confidence: high | medium | low)
  Critical = data loss, can't use core feature, security.
  High     = core feature broken for many, no workaround.
  Medium   = feature degraded or broken with a workaround.
  Low      = cosmetic, rare, or minor annoyance.
  Reason: <one line — why this level>
  (if confidence: low → add ⚠ verify before creating)

PRIORITY: <Urgent (1) | High (2) | Medium (3) | Low (4)>  (confidence: high | medium | low)
  Maps to Linear's native priority number. Severity and priority are not the
  same: a high-severity bug that only one tester hits once might still be Medium
  priority; a low-severity bug blocking a launch might be Urgent.
  Reason: <one line — why this priority>
  (if confidence: low → add ⚠ verify before creating)

LIKELY CAUSE (GUESS — verify before trusting):  (confidence: high | medium | low)
  <name the file(s) / area, cite file.ext:line from what you actually read;
   define any jargon in plain language. ALWAYS label this a guess.
   If the report + code are too thin to guess, say so rather than inventing one.>
  [INFERRED — my analysis, not observed]
  (if confidence: low → add ⚠ verify before creating)

MISSING / TO CONFIRM:
  - <each UNKNOWN field and unanswered question — what the user should go ask
    the tester (device? browser? logged in? which record? screenshot?)>
```

### 4. Check for duplicates in Linear

Before asking for approval, search Linear for issues with similar keywords. If
a likely duplicate exists, surface it:

> "There's an open ticket MYJ-45: 'Voice drops during step nav' — should I
> create a new ticket or add context to that one?"

Then wait for the user to decide before proceeding.

### 5. Run a self-check, then stop and ask for approval

Before asking, run a quick self-check on your own draft and surface the result
so the reviewer knows what to scrutinize:

```text
Before you decide:
- [ ] Every factual claim has a source (report quote or file:line), or is marked as inference
- [ ] Every unknown is written as UNKNOWN or (?) and appears in MISSING / TO CONFIRM
- [ ] Severity has a one-line justification
- [ ] Priority has a one-line justification
- [ ] A Linear duplicate search was run (or is flagged as skipped, and why)
```

If any item is unchecked, call it out in one line before the draft (e.g.
"⚠ Self-check: severity is inferred from report language — verify before
approving."). If everything passes, say "✓ Self-check passed."

Then end with a single clear question:

> "Want me to create this in Linear, or change anything first?"

Then stop. Do not call any Linear write tool yet.

### 6. Handle edits before creating

If the user requests changes to the draft, apply them and show the full revised
draft again. Ask for approval again. Never create from a partially-shown edit
or infer approval from an edit request alone.

### 7. On approval — discover the workspace, then create

Do NOT hard-code team names, statuses, labels, or projects. Discover them at
runtime:

- List teams. If there is only one, use it. If there are several, ask which
  once and remember the answer for the rest of the session.
- **Project:** Check whether the user's CLAUDE.md specifies a default Linear
  project for bugs (e.g. `Linear bug project: my-app`). If it does, use that.
  If not, list available projects on the team and ask the user to pick one —
  but only ask once per session, then remember the choice.
- List the team's issue statuses and pick the appropriate "untriaged / backlog
  / todo" starting status.
- List the team's labels. If a label matching the severity exists (e.g.
  "Critical", "High"), apply it. Otherwise, put severity only in the
  description.
- Map the draft's PRIORITY to Linear's priority number:
  Urgent = 1, High = 2, Medium = 3, Low = 4.
- Put SUMMARY, STEPS TO REPRODUCE, EXPECTED/ACTUAL, LIKELY CAUSE, and
  MISSING/TO CONFIRM into the issue description as Markdown.
- Create the issue. Report back the issue identifier and URL.

### 8. After creation

Confirm what was created in one line. Do not start the next bug unless the user
pastes one.

## Anti-patterns

- **Inventing detail.** Never fill in repro steps, version numbers, error messages, or environment info that the report didn't provide. Use `UNKNOWN` and put it in MISSING / TO CONFIRM.
- **Inflating severity without justification.** Every severity and priority assignment needs a one-line reason grounded in the report or code evidence. "This seems serious" is not a justification.
- **Treating a partial "looks good" as approval.** Enthusiasm about one field, an edit to another, or no objection to the summary is not permission to create. The skill asks explicitly and waits for an explicit answer.
- **Creating before searching for duplicates.** The duplicate check runs before the approval ask, every time. If the Linear connector is unavailable, the skill stops rather than skipping the check silently.
- **Reporting a ticket as created when the write failed or was skipped.** If the create call didn't succeed, the skill says so and shows what it tried. It never says "Done — ticket created" when it wasn't.
- **Presenting the cause guess as confirmed.** The LIKELY CAUSE field is labeled a guess throughout. It does not migrate into the title, summary, or priority justification as though it were established fact.

## Style notes

- Keep drafts tight. This is a triage tool, not an essay.
- The user may be a non-technical founder or PM. When the cause guess involves
  code, explain the *why* in plain language and define any jargon the first
  time it appears.
- One bug at a time unless the user pastes several at once; then draft each
  separately under its own heading and ask for approval per ticket.
