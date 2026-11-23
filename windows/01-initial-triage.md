# Windows — Initial Triage

Run these from an elevated PowerShell prompt where possible. Native commands are given first;
Sysinternals tool equivalents are noted where they give more detail.

## 1. System basics

```powershell
systeminfo
Get-ComputerInfo | Select OsName, OsVersion, WindowsBuildLabEx
whoami /all
net user
net localgroup administrators
```

## 2. Who's logged in / logged in recently

```powershell
query user                       # current sessions
quser
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} -MaxEvents 50 |
  Select TimeCreated, @{n='User';e={$_.Properties[5].Value}}, @{n='LogonType';e={$_.Properties[8].Value}}
# Event ID 4624 = successful logon, 4625 = failed logon
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 50
```

Logon types worth knowing: `2`=interactive, `3`=network, `4`=batch, `5`=service, `10`=RemoteInteractive
(RDP). A `Type 3` or `Type 10` logon for an account that should never remote in is a red flag.

## 3. User accounts & groups

```powershell
Get-LocalUser | Select Name, Enabled, LastLogon, PasswordLastSet
Get-LocalGroupMember -Group "Administrators"
net user <suspicious-account>    # check when it was created, password age, group membership
```

Red flags: an account not matching the expected baseline, an account in Administrators that shouldn't
be, a disabled-looking account that's actually enabled, `PasswordLastSet` very recent for an account
that shouldn't have changed.

## 4. Running processes & network

```powershell
Get-Process | Sort CPU -Descending | Select -First 20
Get-NetTCPConnection -State Established | Select LocalAddress,LocalPort,RemoteAddress,RemotePort,OwningProcess
Get-Process -Id <pid>             # resolve OwningProcess to a process name
netstat -anob                      # needs admin; shows owning exe per connection

# Processes running from unusual locations
Get-Process | Select Name, Path | Where-Object { $_.Path -match '\\Temp\\|\\AppData\\|\\ProgramData\\|\\Users\\Public\\' }
```

If Sysinternals is available (see `tools/tool-list.md`): `Process Explorer` and `TCPView` give a much
faster visual read of the same information, plus VirusTotal hash lookups for suspicious binaries
(note: VT lookups need internet, which may not be available/allowed in the exercise environment).

## 5. Quick filesystem check

```powershell
Get-ChildItem C:\Users\*\AppData\Roaming -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-2) }
Get-ChildItem C:\Windows\Temp, $env:TEMP -Force | Sort LastWriteTime -Descending | Select -First 30
```

## 6. What's scheduled to run

Covered in depth in `02-persistence-hunting.md`, first pass:

```powershell
Get-ScheduledTask | Where-Object {$_.State -ne 'Disabled'} | Select TaskName, TaskPath, State
schtasks /query /fo LIST /v | findstr /i "TaskName Run"
```

## Triage priority order

1. Who can log in / who's an admin (accounts & groups) — attacker's foothold.
2. What's listening / connected out (network) — active backdoor/C2.
3. What's scheduled or auto-starting (tasks, services, run keys) — persistence.
4. What's changed on disk recently — scope of compromise.
