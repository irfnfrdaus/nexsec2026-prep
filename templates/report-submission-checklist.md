# Report Submission Checklist

Run through this before submitting anything, Day 1 or Day 2. It's built directly from what the
guideline says gets reviewed — the point is to catch gaps while there's still time to fix them.

## Content completeness

- [ ] Does the report identify the actual weakness/intrusion path, not just the symptoms you cleaned up?
- [ ] Does it state whether control was recovered (accounts secured, access removed, config fixed) —
      not just that something was "made to look clean"?
- [ ] Have you confirmed and stated that required services still work after your changes?
- [ ] Is every claim backed by a specific piece of evidence (log line, file, timestamp, command output)
      that someone else could go check?
- [ ] Have you clearly separated what the evidence directly shows from what you're inferring?
- [ ] Are gaps/limitations stated explicitly (cleared logs, missing evidence, inconclusive findings)
      rather than glossed over?

## Team process (also graded)

- [ ] Does the report/findings log reflect how work was divided across the team?
- [ ] Is there a record (handover notes, findings log timestamps) that shows the team kept track of
      progress rather than everyone working in isolation?

## Formatting / hygiene

- [ ] Timestamps are consistent (same timezone throughout, stated explicitly if relevant).
- [ ] Evidence references are specific (file path + line, not just "the log showed something off").
- [ ] No leftover placeholder text from the templates (e.g. blank table rows, "___________").
- [ ] Spelling/grammar pass — a report that's hard to follow undermines findings that are otherwise solid.

## Last check before hitting submit

- [ ] Does this match the submission format and deadline the organizers actually specified? (Don't
      assume — if it was given on the ticketing system or during the briefing, double-check against
      that, not against this repo's templates alone.)
- [ ] Has at least one other teammate read it over, even briefly? A second pair of eyes catches gaps
      the person who did the work is too close to see.

## If you're short on time

Prioritize in this order: (1) root cause stated with evidence, (2) remediation actions listed clearly,
(3) required-services verification stated, (4) everything else. A short, accurate report beats a long
one that runs out of evidence to back its claims.
