# NEXSEC2026 — Rules Quick Reference

Read this before the event. It's a summary of the official "NEXSEC2026 CTF Exercise Guideline for
Professional" (PI-RB-01) — if anything here conflicts with that document, the official document wins.

## What this exercise actually is

This is **not** a flag-hunting boot2root CTF. It is a **Red and Blue Team Scenario Exercise**:

- **Day 1 (25 Nov):** You're assigned Linux hosts, Windows hosts, and/or web applications that have
  already been compromised in a scenario. The task is incident response — figure out what happened,
  recover control, fix the weaknesses, and keep required services running. Changes you make **stay
  made** — there's no reset.
- **Day 2 (26 Nov):** Pure digital forensics on evidence you haven't seen before. No live systems.

There is no mention of "flags" in the official guideline. Treat deliverables as **findings + a
defensible report**, not a flag string.

## Hard rules (don't break these)

- **Stay in scope.** Only touch systems assigned to your team, only through the provided access path.
  Never target another team's environment or the organiser's own infrastructure.
- **Don't reset, rebuild, or wipe assigned systems.** A destroyed system will not be replaced.
- **No DoS** unless an organiser explicitly authorises it for a stated part of the exercise.
- **Don't bring in your own equipment/tunnels/accounts** to reach an assigned system.
- **Forensic evidence (Day 2):** work on copies, never alter the original. Only use evidence/resources
  the organisers actually gave you — don't go looking for outside context or answers.
- **No AI in any form on 25–26 Nov**, including checking a single command, explaining a log, or
  passing any exercise material into an AI tool on your own machine. The only exception is a query
  submitted through the organisers' ticketing system, which costs your team points. This repo is
  **pre-event study material only** — don't open it on an AI tool, and don't feed live exercise
  material into anything AI-based once the event starts.
- **Don't share credentials, access, or solutions with another team.**
- **Report, don't troubleshoot alone**, if access fails or you see something that doesn't look like it
  belongs to your team — stop and flag it to the organisers.

## What's graded (per the guideline)

- Whether you identified the actual weakness/intrusion path, not just symptoms.
- Whether you **recovered control** (accounts, access, config) — not just made it "look" clean.
- Whether required services **still work** after your changes.
- Whether your findings/report are things a third party could verify from the evidence.
- How you divided work and kept records as a team.

Keep this in mind while using the rest of this repo: the goal is methodology you can execute fast and
explain clearly, not pre-baked answers — the organisers explicitly said not to assume the task content
in advance.
