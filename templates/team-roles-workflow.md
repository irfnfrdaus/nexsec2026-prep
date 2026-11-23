# Team Roles & Workflow

The guideline explicitly lists "divide the work within the team and make technical decisions under
time pressure" as part of the standard you're assessed against. Agree on this structure (or your own
version of it) *before* the event, not during.

## Suggested role split (adjust to your team size)

| Role | Responsibility |
|---|---|
| **Lead / coordinator** | Tracks overall time budget, decides priority order across assigned systems, is the point of contact for the ticketing system if something needs organizer clarification. |
| **Linux/Windows triage** | Runs initial triage + persistence hunting on assigned hosts (can be split by OS if you have enough people). |
| **Web app specialist** | Focuses on webshell detection, access log analysis, CMS-specific checks if a web app is assigned. |
| **Forensics/log analyst** | Builds the timeline, cross-references logs across systems, keeps the evidence/findings log up to date in real time. |
| **Scribe / report writer** | Owns the findings log and final report structure continuously, rather than trying to reconstruct everything at the end — this directly supports "records sufficient for another team member to continue the work." |

For a small team (2–3 people), merge roles but **keep the scribe function distinct** — someone should
be writing things down in real time no matter how few people you have, since reconstructing a timeline
from memory at the end costs more time than it seems like it will.

## Suggested Day 1 workflow

1. **First 10–15 min:** whole team does quick triage together on whatever's assigned, to get a shared
   picture before splitting up (use `cheatsheets/` for speed).
2. **Split by system/specialty** once the picture is clear — don't duplicate effort on the same host.
3. **Checkpoint every 30–45 min:** quick verbal sync — what's found, what's in progress, anything that
   changes priority (e.g. finding active C2 should immediately become the top priority over a cosmetic
   misconfiguration).
4. **Before any break:** use `templates/handover-notes-template.md` so work doesn't stall or get
   duplicated when people step away.
5. **Last 30 min of each session:** stop finding new things, start consolidating the findings log and
   confirming required services are still up — the guideline grades whether things still work, not
   just whether you found problems.

## Suggested Day 2 workflow (forensics — no live systems)

1. Confirm what evidence you've been given and distribute pieces by type (disk image, memory image,
   pcap, logs) to whoever has the relevant tool skillset.
2. Build individual findings first, then merge into one team timeline — don't try to build one shared
   timeline live with everyone editing it simultaneously; that's slower.
3. Assign one person to own the final report structure early, so findings get written in a consistent
   format as they come in rather than needing reformatting at the end.
4. Leave real time at the end for the "evidence-to-conclusion mapping" step
   (`templates/forensic-report-template.md` section 6) — this is often rushed and is exactly what
   separates a defensible report from a guess.

## Decision-making under time pressure

- Prioritize findings that affect **control of the system** (admin access, backdoors) over cosmetic
  issues — the guideline explicitly says to prioritize this way.
- If the team disagrees on priority, the coordinator makes the call and the team moves — don't burn
  time on a prolonged debate about priority.
- If something is unclear or ambiguous about scope or the task itself, use the ticketing system rather
  than guessing — the guideline repeatedly asks for this rather than participants filling gaps
  themselves.
