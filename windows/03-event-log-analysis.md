# Windows — Event Log Analysis

## Key logs

| Log | Access | What it holds |
|---|---|---|
| Security | `Get-WinEvent -LogName Security` | Logons/logoffs, privilege use, account/group changes, object access (if auditing enabled) |
| System | `Get-WinEvent -LogName System` | Service start/stop, driver loads, system-level events |
| Application | `Get-WinEvent -LogName Application` | Application errors/crashes — sometimes reveals exploitation attempts |
| PowerShell Operational | `Microsoft-Windows-PowerShell/Operational` | Script block logging (if enabled) — can show exact attacker commands |
| Windows Defender Operational | `Microsoft-Windows-Windows Defender/Operational` | AV detections (even if the attacker later disabled/evaded Defender, earlier alerts may still be here) |

## Event IDs worth memorizing

| ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4634 | Logoff |
| 4648 | Logon using explicit credentials (often indicates lateral movement / runas) |
| 4672 | Special privileges assigned to new logon (admin-equivalent logon) |
| 4720 | User account created |
| 4722 | User account enabled |
| 4724 | Password reset attempt |
| 4728 / 4732 / 4756 | Member added to global/local/universal group |
| 4738 | User account changed |
| 4698 | Scheduled task created |
| 4697 | Service installed |
| 7045 | New service installed (System log) |
| 1102 | Audit log cleared (**always a major red flag**) |

## Fast queries

```powershell
# All successful logons in the last 2 days, with source
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddDays(-2)} |
  Select TimeCreated, @{n='User';e={$_.Properties[5].Value}}, @{n='SourceIP';e={$_.Properties[18].Value}}

# Failed logons grouped by source — brute force detection
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} |
  Group-Object { $_.Properties[19].Value } | Sort Count -Descending

# Was the audit log cleared? (attacker covering tracks)
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=1102}

# New services installed
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7045}

# PowerShell command history (if script block logging was on)
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -FilterXPath "*[System[(EventID=4104)]]" |
  Select TimeCreated, Message | Format-List
```

## Building a timeline

1. Anchor on a known reference time (scenario start / when you received the host).
2. Pull 4624/4625/4648/4672 in order — establishes the access pattern.
3. Overlay 4720/4732/4697/4698 (account & persistence changes) — establishes what the attacker did
   once in.
4. Check for 1102 (log clearing) — if present, the true start of the intrusion may be earlier than
   your earliest visible event; say so explicitly in your report rather than assuming the log is
   complete.
5. Cross-reference with file system timestamps and `$MFT`/`$UsnJrnl` if you have forensic tooling
   (Day 2) — live Event Logs alone can be incomplete, especially if cleared.

## Caveats

- An attacker with admin rights can clear the Security log (event 1102) or disable auditing entirely —
  absence of evidence isn't evidence of absence. Note gaps explicitly.
- Logon Type matters more than the fact of a logon: Type 3 (network) from an internal admin share and
  Type 10 (RDP) from an unexpected source are very different findings.
- Script block logging (4104) only exists if it was enabled beforehand — don't assume you'll have it.
