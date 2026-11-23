# NEXSEC2026 Prep — Red/Blue Team Exercise & Forensics Reference

Prep repo for NEXSEC2026 (MITEC, 25–26 Nov 2026). **Read `00-rules-and-reminders.md` first** — the
official exercise format and rules are different from a standard flag-based boot2root CTF, and the AI
policy applies to this repo too (study material only, pre-event — see that file for details). Then
check `00b-pre-event-logistics-checklist.md` for what to confirm with organizers beforehand.

## Don't know where to start once you're assigned a system? Start here.

- **`01-DAY1-MASTER-RUNBOOK.md`** — a single linear, step-by-step sequence for the live Day 1 exercise,
  regardless of whether you're assigned a Linux host, Windows host, or web app. Follow it top to bottom;
  it tells you exactly which other file to open at each step.
- **`02-DAY2-FORENSICS-RUNBOOK.md`** — the same idea for Day 2's evidence-only forensics work.

Everything else in this repo is the detailed reference material those two runbooks point you to.

## Quick keyword index

Use Ctrl+F / your editor's search on this table to jump to the right file fast during study, or to
remember where something lives.

| Looking for... | Go to |
|---|---|
| Rules, scoring, what's allowed | `00-rules-and-reminders.md` |
| Pre-event logistics / what to confirm with organizers | `00b-pre-event-logistics-checklist.md` |
| **Step-by-step Day 1 workflow (start here if stuck)** | `01-DAY1-MASTER-RUNBOOK.md` |
| **Step-by-step Day 2 forensics workflow (start here if stuck)** | `02-DAY2-FORENSICS-RUNBOOK.md` |
| One-page printable command references | `cheatsheets/linux-cheatsheet.md`, `cheatsheets/windows-cheatsheet.md`, `cheatsheets/web-cheatsheet.md` |
| Common ports/services quick lookup during recon | `cheatsheets/common-ports-services.md` |
| Linux: first things to check on a box | `linux/01-initial-triage.md` |
| Linux: cron, SSH keys, SUID, systemd persistence | `linux/02-persistence-hunting.md` |
| Linux: auth.log, syslog, journald, bash history | `linux/03-log-analysis.md` |
| Linux: fixing what you found | `linux/04-hardening-checklist.md` |
| Windows: first things to check on a box | `windows/01-initial-triage.md` |
| Windows: Run keys, scheduled tasks, services, WMI | `windows/02-persistence-hunting.md` |
| Windows: Event IDs, Security log, PowerShell logging | `windows/03-event-log-analysis.md` |
| Windows: fixing what you found | `windows/04-hardening-checklist.md` |
| Windows: domain-joined / Active Directory checks | `windows/05-active-directory-enumeration.md` |
| Webshells / backdoors in a web app | `web/01-webshell-detection.md` |
| Apache/Nginx/IIS access log analysis | `web/02-access-log-analysis.md` |
| WordPress, exposed .git, default creds, file upload flaws | `web/03-common-cms-weaknesses.md` |
| Web app hardening | `web/04-hardening-checklist.md` |
| Disk image triage (Day 2) | `forensics/01-disk-imaging-triage.md` |
| Memory forensics / Volatility | `forensics/02-memory-forensics.md` |
| Building a timeline | `forensics/03-timeline-analysis.md` |
| Chain of custody / how to write conclusions | `forensics/04-chain-of-custody.md` |
| Network/PCAP analysis, C2 & exfil detection (Wireshark) | `forensics/05-network-pcap-analysis.md` |
| Nmap, gobuster, general recon | `offensive/01-recon-enumeration.md` |
| SQLi, LFI, file upload exploitation, command injection | `offensive/02-web-exploitation.md` |
| Linux privilege escalation (GTFOBins, SUID, cron) | `offensive/03-linux-privesc.md` |
| Windows privilege escalation (potato exploits, services) | `offensive/04-windows-privesc.md` |
| Hash cracking, brute-forcing, credential dumping | `offensive/05-password-attacks.md` |
| Findings log (fill in live during the exercise) | `templates/findings-log-template.md` |
| Handover notes (fill in when swapping teammates) | `templates/handover-notes-template.md` |
| Forensic report template (Day 2 deliverable) | `templates/forensic-report-template.md` |
| Splitting roles & workflow across your team | `templates/team-roles-workflow.md` |
| Rating findings by severity (what to fix first) | `templates/finding-severity-rating.md` |
| Final checklist before submitting your report | `templates/report-submission-checklist.md` |
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
├── 00b-pre-event-logistics-checklist.md
├── 01-DAY1-MASTER-RUNBOOK.md
├── 02-DAY2-FORENSICS-RUNBOOK.md
├── cheatsheets/
│   ├── linux-cheatsheet.md
│   ├── windows-cheatsheet.md
│   ├── web-cheatsheet.md
│   └── common-ports-services.md
├── linux/
│   ├── 01-initial-triage.md
│   ├── 02-persistence-hunting.md
│   ├── 03-log-analysis.md
│   └── 04-hardening-checklist.md
├── windows/
│   ├── 01-initial-triage.md
│   ├── 02-persistence-hunting.md
│   ├── 03-event-log-analysis.md
│   ├── 04-hardening-checklist.md
│   └── 05-active-directory-enumeration.md
├── web/
│   ├── 01-webshell-detection.md
│   ├── 02-access-log-analysis.md
│   ├── 03-common-cms-weaknesses.md
│   └── 04-hardening-checklist.md
├── forensics/
│   ├── 01-disk-imaging-triage.md
│   ├── 02-memory-forensics.md
│   ├── 03-timeline-analysis.md
│   ├── 04-chain-of-custody.md
│   └── 05-network-pcap-analysis.md
├── offensive/
│   ├── 01-recon-enumeration.md
│   ├── 02-web-exploitation.md
│   ├── 03-linux-privesc.md
│   ├── 04-windows-privesc.md
│   └── 05-password-attacks.md
├── templates/
│   ├── findings-log-template.md
│   ├── handover-notes-template.md
│   ├── forensic-report-template.md
│   ├── team-roles-workflow.md
│   ├── finding-severity-rating.md
│   └── report-submission-checklist.md
└── tools/
    └── tool-list.md
```
