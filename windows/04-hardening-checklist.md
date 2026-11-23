# Windows — Hardening / Remediation Checklist

## Accounts & access

- [ ] Disable/remove unauthorized local accounts (`net user <name> /active:no` or delete per team
      decision — keep a record either way).
- [ ] Remove unauthorized members from Administrators (`Remove-LocalGroupMember`).
- [ ] Reset passwords for compromised or newly-created accounts.
- [ ] Review Group Policy / local security policy for weakened settings (e.g. password policy
      disabled, lockout threshold removed) if time allows: `secpol.msc` or `Get-LocalSecurityPolicy`
      equivalents via `auditpol /get /category:*`.

## Persistence cleanup

- [ ] Remove malicious Run/RunOnce registry keys.
- [ ] Remove/disable malicious scheduled tasks (`Unregister-ScheduledTask -TaskName <name>`).
- [ ] Stop and remove malicious services (`Stop-Service`, then `sc.exe delete <name>` or
      `Remove-Service` on newer PowerShell).
- [ ] Remove WMI event subscriptions if found (`Get-WmiObject ... | Remove-WmiObject`).
- [ ] Clean startup folder entries.

## Service hygiene

- [ ] Confirm every service that's *supposed* to be running still is (`Get-Service | Where Status
      -ne Running` to spot anything unexpectedly stopped).
- [ ] Re-check listening ports / established connections after remediation.
- [ ] If Defender/AV was disabled by the attacker, re-enable it:
      `Set-MpPreference -DisableRealtimeMonitoring $false`.

## Evidence / integrity

- [ ] Export relevant Event Log entries before making changes that might overwrite log history
      (`wevtutil epl Security C:\evidence\security.evtx`).
- [ ] Note hashes/paths of any malicious binaries found before deleting (use `Get-FileHash`).
- [ ] Document every change, in order, with a short justification — your handover note.

## Common “don’t forget” items

- Firewall rules: `Get-NetFirewallRule | Where Enabled -eq True` — did the attacker open an inbound
  rule for a backdoor port?
- Check `hosts` file for DNS hijacking: `C:\Windows\System32\drivers\etc\hosts`.
- RDP settings: is RDP enabled when it shouldn't be? `Get-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections`.
