# Linux — Hardening / Remediation Checklist

Use this after you've investigated. The guideline explicitly grades "did you correct the weakness" and
"does the required service still work" — not just "did you find the problem."

## Accounts & access

- [ ] Remove/lock unauthorized accounts (`passwd -l <user>` or `usermod -L <user>`; don't delete if you
      might need it for evidence — lock instead, unless told otherwise).
- [ ] Remove unauthorized entries from `/etc/sudoers` / `/etc/sudoers.d/`.
- [ ] Rotate passwords for any account known or suspected to be compromised.
- [ ] Remove unauthorized SSH keys from `~/.ssh/authorized_keys`.
- [ ] Review and tighten `/etc/ssh/sshd_config`: `PermitRootLogin no` (or `prohibit-password`),
      `PasswordAuthentication no` if keys are in use, restart sshd after changes (`systemctl restart sshd`)
      — **test that you can still log in via another session before closing your current one.**

## Persistence cleanup

- [ ] Remove malicious cron jobs / systemd timers / services identified in triage.
- [ ] Remove backdoored SUID binaries, or strip the SUID bit (`chmod u-s <file>`) if the binary itself
      is legitimate but was tampered with.
- [ ] Clear `/etc/ld.so.preload` if it was hijacked.
- [ ] Kill malicious processes; confirm they don't respawn.

## Service hygiene

- [ ] Confirm every service that's *supposed* to be running still is: `systemctl status <service>`.
- [ ] Check listening ports again after remediation (`ss -tulpn`) to confirm nothing's still phoning home.
- [ ] If you patched/updated a vulnerable package, confirm the service that depends on it still starts
      cleanly.

## Evidence / integrity

- [ ] Before deleting anything you believe is malicious, copy it aside (e.g. to a `evidence/` folder
      outside the running system, or at least note its hash/path/contents in your findings) — you may
      need to reference it in your report, and deleting your only copy of "the malware" removes your
      proof.
- [ ] Document every change you make, in order, with a one-line justification each. This is your
      handover note and your audit trail.

## Common “don’t forget” items

- Firewall rules: `iptables -L -n -v` / `nft list ruleset` / `ufw status verbose` — did the attacker
  open a port, or did you leave one open while debugging?
- World-writable files in sensitive directories: `find / -xdev -perm -0002 -type f 2>/dev/null`
- `.bash_history` / shell files with suspicious one-liners left behind by the attacker, pointing to
  tools or C2 infrastructure worth noting in your report even after cleanup.
