# Finding Severity Rating Scheme

Use this to tag every entry in your findings log, so the team can see at a glance what to prioritize
when several findings come in from different people/systems at once.

## Ratings

### Critical
Active attacker control is present or could be re-established immediately. Fix first, always.
- Active backdoor/webshell currently reachable.
- Live C2 connection or reverse shell currently open.
- Unauthorized account with admin/root-equivalent access, currently usable.
- Cleared/tampered audit logs (indicates active cover-up, treat with urgency even though it doesn't
  grant access itself).

### High
Persistence mechanism that would let the attacker regain control, but not actively in use right now.
- Dormant cron job/scheduled task/service that would re-establish access if triggered.
- SSH key or sudoers entry granting access, not currently being used but still present.
- Credentials exposed that could be used to regain access (plaintext passwords, reusable hashes).
- Webshell present but not currently being accessed in logs.

### Medium
A real weakness that contributed to or could contribute to compromise, but doesn't itself grant
ongoing access.
- Outdated software with a known vulnerability, not yet confirmed exploited.
- Misconfiguration that weakens security posture (world-writable files, weak firewall rules) without
  direct evidence of exploitation.
- Exposed sensitive file (`.git`, `.env`) without confirmed evidence it was accessed.

### Low
Hygiene issues worth fixing and reporting, but not plausibly connected to the incident itself.
- Verbose error messages / directory listing enabled.
- Outdated software with no known relevant vulnerability.
- Minor deviations from hardening best practice.

## How to use this

1. Tag each finding in `templates/findings-log-template.md` with one of the four ratings as you log it.
2. Remediate in severity order: all Critical findings before moving to High, etc. — unless fixing a
   lower-severity item is required first to safely fix a higher one (note that dependency explicitly if
   it applies).
3. In the final report, lead with Critical/High findings — these are what most directly answer "did you
   identify the weakness and recover control," which the guideline weights heavily.
4. Don't skip logging Low findings entirely — they still demonstrate thoroughness, just don't let them
   consume time that Critical/High findings still need.

## A note on judgment calls

These definitions won't cleanly cover every situation — if a finding genuinely sits between two
ratings, pick the higher one when the decision affects what gets fixed first, and discuss it with the
team rather than debating it alone against the clock.
