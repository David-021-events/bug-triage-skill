# Walkthrough: bug-triage in action

An illustrative end-to-end run showing how the skill handles a realistic, incomplete
bug report — from intake to a created Linear issue. (The file path and ticket IDs
below are fictional, for illustration only.)

---

## 1. The incoming report

The user pastes a forwarded tester message:

> "hey just fyi the export thing is broken again. tried to download my report as
> CSV and nothing happened, just sat there. was doing a big date range like all of
> last year. didn't get an error or anything. this was maybe 20 mins ago. using
> chrome. – Priya"

---

## 2. The resulting draft

After reading the code areas most likely involved, the skill produces:

```text
TITLE: CSV export silently hangs for large date ranges (no error shown)

SUMMARY: A CSV export over a large date range appears to start but produces no
  download and no error — in Chrome. ("nothing happened, just sat there",
  "using chrome")
  [OBSERVED — from the report]

STEPS TO REPRODUCE (for a human to run, not confirmed by me):
  1. Open the report export screen (?)   ← exact screen not stated
  2. Select a large date range — reporter said "all of last year"
  3. Click export as CSV ("download my report as CSV")
  4. Observe: no download, no error ("nothing happened", "didn't get an error")
  [OBSERVED where cited; (?) where not]

EXPECTED vs ACTUAL:
  Expected: a CSV downloads, or a clear error is shown if it can't.
  Actual:   nothing happens; the UI sits with no feedback. ("just sat there")
  [OBSERVED — from the report]

SEVERITY: Medium  (confidence: medium)
  Reason: feature is non-functional for the reported input, but no data loss or
  security impact, and a smaller date range may be a workaround.

PRIORITY: Medium  (confidence: low)  ⚠ verify before creating
  Reason: single reporter, impact breadth unknown — escalate if large exports
  are a primary workflow or more reports arrive.

LIKELY CAUSE (GUESS — verify before trusting):  (confidence: low)  ⚠ verify before creating
  src/exports/csv-exporter.ts:212 — the chunked-fetch step has no timeout and the
  outer await has no error handler, so a query that exceeds the DB statement
  timeout would hang with nothing surfaced to the UI. ("statement timeout" = the
  database giving up on a slow query.)
  [INFERRED — my analysis, not observed]

MISSING / TO CONFIRM:
  - OS and app version (UNKNOWN — not in report)
  - Account type / data volume (affects whether a timeout is hit)
  - Does it reproduce with a smaller date range?
  - Server-side logs around the report time (~20 min before filing)
  - Whether the csv-exporter.ts:212 timeout hypothesis is correct (needs repro
    or a log check)
```

---

## 3. Duplicate check

The skill searches Linear ("export", "CSV", "hang", "timeout", "silent") and surfaces a candidate:

> **Candidate:** `BUG-318 — CSV export fails for enterprise accounts` (status: In Progress).
> It's scoped to enterprise accounts and blames a row-count limit — may or may not be the same bug.
> **Your call:** same bug, a related variant, or a separate issue?

---

## 4. Pre-approval self-check, then the ask

```text
Before you decide:
- [x] Every factual claim has a source (report quote or file:line), or is marked as inference
- [x] Every unknown is written as UNKNOWN or (?) and appears in MISSING / TO CONFIRM
- [x] Severity has a one-line justification
- [x] Priority has a one-line justification
- [x] A Linear duplicate search was run (candidate surfaced above)
```

> ✓ Self-check passed. Note: priority and likely cause are **low confidence** — worth
> a look before approving.
>
> **Want me to create this in Linear, or change anything first?**
> (Also — same as BUG-318, related, or separate?)

---

## 5. The approval exchange

**User:**

> Separate issue — BUG-318 is enterprise only, this reporter is on the free plan.
> Bump severity to High; we shipped a loading spinner last release, so a silent hang
> means the spinner is broken too. Go ahead and create it.

**Skill:**

> Updating severity to High (the missing spinner makes this a regression) and noting
> free-plan scope. Filing as a separate issue, not linked to BUG-318. Creating now…

---

## 6. Post-creation confirmation

> ✓ Created **BUG-412 — CSV export silently hangs for large date ranges (no error shown)**
> — Triage status, Priority: Medium. https://linear.app/yourteam/issue/BUG-412

(Priority stayed Medium even though severity went to High — severity is impact, priority is urgency.)
