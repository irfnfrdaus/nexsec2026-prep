# Day 1 Master Runbook — "I don't know where to start" → follow this

This is the single linear path through the rest of the repo. When you're handed a system and don't
know what to do first, start here, follow the steps in order, and branch out to the detailed guides
only when a step tells you to.

## 0. Before you touch anything (2 min)

- [ ] Confirm what you've been given: hostname/IP, system type (Linux/Windows/web app), any
      credentials or access path provided, and what the task is actually asking for (recover control?
      investigate only? both?). If this isn't clear, use the ticketing system — don't guess.
- [ ] Open `templates/findings-log-template.md` — copy it for this system, keep it open the whole time.
- [ ] Have the matching cheat sheet open (`cheatsheets/linux-cheatsheet.md`,
      `cheatsheets/windows-cheatsheet.md`, or `cheatsheets/web-cheatsheet.md`).

## 1. Triage (10–15 min) — "what does this system look like right now?"

Follow the full steps in:
- Linux → `linux/01-initial-triage.md`
- Windows → `windows/01-initial-triage.md`
- Web app layered on either → also skim `web/01-webshell-detection.md` triage section

**Goal of this step:** a rough picture of running processes, network connections, logged-in users, and
anything obviously wrong. Write down anything odd in the findings log, even if you don't understand it
yet — you'll come back to it.

## 2. Accounts & access check (5 min)

- Who can log in? Who's an admin/root-equivalent? Any accounts that don't belong?
- Linux: `linux/01-initial-triage.md` section 3, `linux/02-persistence-hunting.md` section 1 (SSH).
- Windows: `windows/01-initial-triage.md` section 3.

This matters early because it tells you whether the attacker can still get back in — closing this is
often the single highest-value action you can take.

## 3. Persistence hunt (15–20 min) — "how does the attacker stay in?"

Go through the full checklist, don't stop at the first thing found:
- Linux → `linux/02-persistence-hunting.md` (cron, SSH keys, systemd, SUID, PAM, LD_PRELOAD)
- Windows → `windows/02-persistence-hunting.md` (Run keys, scheduled tasks, services, WMI)
- If domain-joined → also `windows/05-active-directory-enumeration.md`
- If a web app is present → `web/01-webshell-detection.md`

Log every mechanism found, with evidence (command + output), in your findings log.

## 4. Log analysis (10–15 min) — "build the story of what happened"

- Linux → `linux/03-log-analysis.md`
- Windows → `windows/03-event-log-analysis.md`
- Web app → `web/02-access-log-analysis.md`
- If a packet capture or live traffic is relevant → `forensics/05-network-pcap-analysis.md`

**Goal:** find the earliest anomalous event. That's your entry point / root cause. Write it down
explicitly: timestamp, source, what happened, why it's anomalous.

## 5. Identify the entry point (5–10 min)

By now you should be able to answer: **how did the attacker get in?** Common answers:
- Weak/default credentials → see `offensive/05-password-attacks.md` for what weak looks like
- A web app vulnerability → `web/03-common-cms-weaknesses.md`
- An exposed/misconfigured service → whatever you found in triage
- A phished/compromised legitimate account → accounts check in step 2

If you genuinely can't find an entry point after steps 1–4, that's a valid (if incomplete) finding —
write what you found and what you couldn't determine, rather than guessing.

## 6. Remediate (15–20 min)

Work through the matching checklist:
- Linux → `linux/04-hardening-checklist.md`
- Windows → `windows/04-hardening-checklist.md`
- Web app → `web/04-hardening-checklist.md`

**Before you delete/remove anything malicious:** copy it aside or note its hash/path/content in your
findings log — you need it as evidence for the report.

## 7. Verify (5 min) — don't skip this

- [ ] Confirm every required service is still running (`systemctl status`, `Get-Service`, or just
      load the web app in a browser).
- [ ] Re-check for the persistence mechanism you removed — confirm it doesn't come back.
- [ ] Re-check listening ports/processes one more time.

## 8. If you have time left

- Rate your findings using `templates/finding-severity-rating.md` so the team can see what matters most.
- Cross-check `offensive/01-05-*.md` in case there's an offensive component to the task you haven't
  addressed yet (e.g. "demonstrate the vulnerability" rather than just "fix it").
- Update the handover notes (`templates/handover-notes-template.md`) if you're about to switch
  systems or take a break.

## When you're truly stuck

1. Re-read what the task is actually asking — half of "not knowing how to solve it" is not having
   re-read the actual prompt/requirements closely enough.
2. Go back to triage (step 1) and look again — something is usually visible that gets missed the
   first pass through.
3. Ask the team — someone else may have seen the same pattern on a different assigned system.
4. If genuinely blocked on an environment/access issue (not a "we don't understand the vulnerability"
   issue), use the ticketing system.
5. Write down what you tried and what you found even if incomplete — partial, evidenced findings
   score better than nothing, and the guideline grades the process, not just a clean final answer.
