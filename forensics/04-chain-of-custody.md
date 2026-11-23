# Forensics — Chain of Custody & Evidence Handling

The guideline is explicit: *"Leave original evidence unchanged and work on copies. State only what the
evidence supports."* and *"Original evidence must not be altered or destroyed... participants must not
look outside the material provided for data, context or an answer."* Treat this as a hard rule, not a
suggestion — it's likely part of how Day 2 is scored.

## Rules to follow

1. **Never run analysis tools directly against the original evidence file.** Hash it, copy it, verify
   the copy, and work on the copy.
2. **Only use evidence and resources the organizers actually gave you.** Don't search the internet for
   "what this scenario is probably about," don't guess context not present in the material, and don't
   import outside IOC lists as if they were part of the evidence.
3. **Keep a running log** of every action taken: what file, what tool, what command, what you found,
   and when. This is your chain of custody.

## Minimal evidence log template

```
Evidence ID: ______________________
Original filename: ______________________
Original SHA256: ______________________
Received from: Organisers, [time/date]
Copy made: [time/date], SHA256 of copy: ______________________ (must match original)

| Timestamp | Analyst | Action | Tool used | Finding |
|---|---|---|---|---|
|  |  |  |  |  |
```

## Writing conclusions the guideline will accept

- Every claim in your final write-up should trace back to a specific piece of evidence you can point
  to (a log line, a timestamp, a file, a registry key).
- Distinguish clearly between **what the evidence shows** and **what you infer from it.** E.g.: "The
  access log shows a POST request to `/uploads/shell.php` at 14:32:07 from 10.0.0.5 (evidence). This is
  consistent with the webshell being uploaded via the image-upload form, given the earlier GET request
  to that form from the same IP at 14:31:50 (inference, stated as such)."
- If evidence is incomplete, missing, or contradictory (e.g. a cleared log), say so rather than
  filling the gap with a guess.

## Why this matters for scoring

Section 6.4.2 of the guideline says the organizers review work against the requirement set, and
Section 4 says the standard includes "record conclusions that a third party could verify." A
well-sourced, honestly-scoped report will hold up to that kind of review far better than a confident
narrative that outruns the evidence.
