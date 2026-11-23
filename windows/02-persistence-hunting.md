# Windows — Persistence Hunting

Windows has many more persistence mechanisms than Linux. Work through this list methodically —
don't stop at the first thing you find.

## 1. Registry Run keys (classic, most common)

```powershell
Get-ItemProperty 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty 'HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\RunOnce'
# Also check the WOW6432Node equivalents on 64-bit systems:
Get-ItemProperty 'HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run' -ErrorAction SilentlyContinue
```

Red flags: an entry pointing to a script/exe in `%TEMP%`, `%APPDATA%`, `C:\Users\Public`, or a
PowerShell one-liner with `-enc` (base64-encoded command) or `-w hidden`.

## 2. Scheduled tasks

```powershell
Get-ScheduledTask | Where-Object {$_.State -ne 'Disabled'} |
  ForEach-Object { Get-ScheduledTaskInfo -TaskName $_.TaskName -TaskPath $_.TaskPath }
Get-ScheduledTask | Select TaskName, TaskPath, State, Actions |
  ForEach-Object { $_.Actions | Select @{n='Task';e={$using:_.TaskName}}, Execute, Arguments }
schtasks /query /fo LIST /v
```

Inspect any task whose `Execute`/`Arguments` references PowerShell, `cmd.exe`, `mshta.exe`,
`rundll32.exe`, `regsvr32.exe`, `certutil.exe`, or `wscript`/`cscript` — these are frequently abused
as "living off the land" execution wrappers.

## 3. Services

```powershell
Get-WmiObject win32_service | Select Name, DisplayName, PathName, StartMode, State |
  Where-Object { $_.PathName -notmatch '^"?C:\\Windows\\(System32|SysWOW64)' }
# ^ services whose binary path is outside the normal Windows system folders deserve a closer look
```

Red flags: a service name mimicking a legitimate one (typosquatting, e.g. "Windows Udpate Service"),
a `PathName` pointing into `AppData`/`Temp`/`Public`, unquoted service paths with spaces (can be a
privesc vector too — see `offensive/04-windows-privesc.md`).

## 4. WMI event subscriptions (fileless persistence — check if time allows)

```powershell
Get-WmiObject -Namespace root\subscription -Class __EventFilter
Get-WmiObject -Namespace root\subscription -Class __EventConsumer
Get-WmiObject -Namespace root\subscription -Class __FilterToConsumerBinding
```

If any of these return results beyond a clean baseline, treat it as a serious, stealthy persistence
mechanism — WMI subscriptions survive reboots and don't show up in the Run keys or Task Scheduler.

## 5. Startup folders

```powershell
Get-ChildItem "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
Get-ChildItem "C:\Users\*\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
```

## 6. Logon scripts / Group Policy

```powershell
Get-ItemProperty 'HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon' | Select Shell, Userinit
# Userinit should be exactly "C:\Windows\system32\userinit.exe,"  — anything appended is a red flag
```

## 7. New/modified local accounts and group membership

```powershell
net user
Get-LocalGroupMember -Group "Administrators"
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4720} -MaxEvents 20   # account created
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4732} -MaxEvents 20   # member added to local group
```

## 8. DLL hijacking / AppInit_DLLs (time permitting)

```powershell
Get-ItemProperty 'HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Windows' | Select AppInit_DLLs
```

## Remediation checklist

- [ ] Remove the malicious Run key / scheduled task / service / WMI subscription.
- [ ] Disable or remove any unauthorized local account; remove from Administrators if added there.
- [ ] Reset passwords for compromised accounts.
- [ ] Kill malicious processes and confirm they don't respawn via a task/service you missed.
- [ ] Re-check required services are still running after cleanup.
- [ ] Note every IOC (file path, registry key, task name, IP/domain) in your findings log.
