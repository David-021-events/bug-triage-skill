---
name: bug-triage
version: "1.0.0"
description: >
  Triage a raw bug report (pasted from email, WhatsApp, or anywhere) into a
  structured draft and, ONLY after the user approves, create a Linear issue.
  Trigger when the user pastes a bug report, forwards a tester complaint, or
  says "triage this", "log this bug", or "turn this into a ticket". Produces a
  draft with title, repro steps, severity, priority, a code-area hypothesis,
  and a list of what's missing — then waits for explicit confirmation before
  writing anything to Linear.
repository: https://github.com/David-021-events/bug-triage-skill
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

```
TITLE
  A short, specific, searchable title. Describe the symptom, not the guess.
  Good: "Cook Mode voice command 'next' skips two steps on iOS Safari"
  Bad:  "voice bug"

SUMMARY
  1-2 plain sentences describing what the tester observed.

STEPS TO REPRODUCE
  Numbered steps, inferred from the report. Where a step is unknown, write
  the step and mark it (?) so it's obvious what needs confirming.

EXPECTED vs ACTUAL
  Expected: what should happen.
  Actual:   what the tester saw.

SEVERITY  — how bad is it if it happens? With a one-line reason.
  Use: Critical / High / Medium / Low.
  Critical = data loss, can't use core feature, security.
  High     = core feature broken for many, no workaround.
  Medium   = feature degraded or broken with a workaround.
  Low      = cosmetic, rare, or minor annoyance.

PRIORITY — how soon should it be fixed? With a one-line reason. Maps to
  Linear's native scale:
    Urgent (1) / High (2) / Medium (3) / Low (4)
  Severity and priority are not the same: a high-severity bug that only one
  tester hits once might still be Medium priority.

LIKELY CAUSE (GUESS — verify before trusting)
  Name the file(s) or area most likely involved and why, based on what you
  read in the repo. ALWAYS label this a guess. Explain in plain language —
  define any technical terms the first time they appear. If the report is too
  thin to guess, say so rather than inventing one.

MISSING / TO CONFIRM
  A bullet list of what the report doesn't tell us and what the user should
  go ask the tester (device? browser? logged in? which record? screenshot?).
```

### 4. Check for duplicates in Linear

Before asking for approval, search Linear for issues with similar keywords. If
a likely duplicate exists, surface it:

> "There's an open ticket MYJ-45: 'Voice drops during step nav' — should I
> create a new ticket or add context to that one?"

Then wait for the user to decide before proceeding.

### 5. Stop and ask for approval

End the draft with a single clear question:

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

## Style notes

- Keep drafts tight. This is a triage tool, not an essay.
- The user may be a non-technical founder or PM. When the cause guess involves
  code, explain the *why* in plain language and define any jargon the first
  time it appears.
- One bug at a time unless the user pastes several at once; then draft each
  separately under its own heading and ask for approval per ticket.
