# NEXSEC2026 Prep — Red/Blue Team Exercise & Forensics Reference

Prep repo for NEXSEC2026 (MITEC, 25–26 Nov 2026). **Read `00-rules-and-reminders.md` first** — the
official exercise format and rules are different from a standard flag-based boot2root CTF, and the AI
policy applies to this repo too (study material only, pre-event — see that file for details).

## Quick keyword index

Use Ctrl+F / your editor's search on this table to jump to the right file fast during study, or to
remember where something lives.

| Looking for... | Go to |
|---|---|
| Rules, scoring, what's allowed | `00-rules-and-reminders.md` |
| Linux: first things to check on a box | `linux/01-initial-triage.md` |
| Linux: cron, SSH keys, SUID, systemd persistence | `linux/02-persistence-hunting.md` |
| Linux: auth.log, syslog, journald, bash history | `linux/03-log-analysis.md` |
| Linux: fixing what you found | `linux/04-hardening-checklist.md` |
| Windows: first things to check on a box | `windows/01-initial-triage.md` |
| Windows: Run keys, scheduled tasks, services, WMI | `windows/02-persistence-hunting.md` |
| Windows: Event IDs, Security log, PowerShell logging | `windows/03-event-log-analysis.md` |
| Windows: fixing what you found | `windows/04-hardening-checklist.md` |
| Webshells / backdoors in a web app | `web/01-webshell-detection.md` |
| Apache/Nginx/IIS access log analysis | `web/02-access-log-analysis.md` |
| WordPress, exposed .git, default creds, file upload flaws | `web/03-common-cms-weaknesses.md` |
| Web app hardening | `web/04-hardening-checklist.md` |
| Disk image triage (Day 2) | `forensics/01-disk-imaging-triage.md` |
| Memory forensics / Volatility | `forensics/02-memory-forensics.md` |
| Building a timeline | `forensics/03-timeline-analysis.md` |
| Chain of custody / how to write conclusions | `forensics/04-chain-of-custody.md` |
| Nmap, gobuster, general recon | `offensive/01-recon-enumeration.md` |
| SQLi, LFI, file upload exploitation, command injection | `offensive/02-web-exploitation.md` |
| Linux privilege escalation (GTFOBins, SUID, cron) | `offensive/03-linux-privesc.md` |
| Windows privilege escalation (potato exploits, services) | `offensive/04-windows-privesc.md` |
| Hash cracking, brute-forcing, credential dumping | `offensive/05-password-attacks.md` |
| Findings log (fill in live during the exercise) | `templates/findings-log-template.md` |
| Handover notes (fill in when swapping teammates) | `templates/handover-notes-template.md` |
| Forensic report template (Day 2 deliverable) | `templates/forensic-report-template.md` |
| Tools to install beforehand + download links | `tools/tool-list.md` |

## Suggested prep order

1. Read `00-rules-and-reminders.md` so everyone on the team has the same understanding of scope and
   grading.
2. Install everything in `tools/tool-list.md` and get comfortable with the syntax — you won't be able
   to ask an AI for help with commands once the event starts.
3. Walk through the Linux and Windows triage + persistence guides on a practice VM of your own (or a
   HTB/TryHackMe box) until the commands are second nature.
4. Do the same for the web app guides.
5. Run through the forensics guides against a sample disk/memory image (many are publicly available
   for practice — e.g. through DFIR training CTFs) so Autopsy/Volatility aren't unfamiliar on the day.
6. Print or memorize the templates in `templates/` — you'll want a findings log and a report structure
   ready to go immediately rather than improvising one mid-exercise.
7. Skim `offensive/` as a backup reference in case the actual tasks lean more exploitation-heavy than
   the official guideline suggests.

## Repo structure

```
.
├── 00-rules-and-reminders.md
├── linux/
│   ├── 01-initial-triage.md
│   ├── 02-persistence-hunting.md
│   ├── 03-log-analysis.md
│   └── 04-hardening-checklist.md
├── windows/
│   ├── 01-initial-triage.md
│   ├── 02-persistence-hunting.md
│   ├── 03-event-log-analysis.md
│   └── 04-hardening-checklist.md
├── web/
│   ├── 01-webshell-detection.md
│   ├── 02-access-log-analysis.md
│   ├── 03-common-cms-weaknesses.md
│   └── 04-hardening-checklist.md
├── forensics/
│   ├── 01-disk-imaging-triage.md
│   ├── 02-memory-forensics.md
│   ├── 03-timeline-analysis.md
│   └── 04-chain-of-custody.md
├── offensive/
│   ├── 01-recon-enumeration.md
│   ├── 02-web-exploitation.md
│   ├── 03-linux-privesc.md
│   ├── 04-windows-privesc.md
│   └── 05-password-attacks.md
├── templates/
│   ├── findings-log-template.md
│   ├── handover-notes-template.md
│   └── forensic-report-template.md
└── tools/
    └── tool-list.md
```
