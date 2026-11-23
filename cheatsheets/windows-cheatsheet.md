# Windows Cheat Sheet (print this one page)

## Triage
```powershell
systeminfo; whoami /all
query user; quser
Get-Process | Sort CPU -Descending | Select -First 20
Get-NetTCPConnection -State Established
```

## Accounts
```powershell
Get-LocalUser | Select Name, Enabled, LastLogon, PasswordLastSet
Get-LocalGroupMember -Group "Administrators"
```

## Persistence (check ALL of these)
```powershell
Get-ItemProperty 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ScheduledTask | Where-Object {$_.State -ne 'Disabled'}
Get-WmiObject win32_service | Where-Object { $_.PathName -notmatch '^"?C:\\Windows\\' }
Get-WmiObject -Namespace root\subscription -Class __EventFilter
Get-ChildItem "C:\Users\*\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
```

## Event logs (key IDs: 4624 logon, 4625 fail, 4672 admin logon, 4720 new user, 4732 group add, 1102 log cleared)
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} -MaxEvents 50
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 50
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=1102}   # audit log cleared = major red flag
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7045}     # new service installed
```

## Privesc (if offensive-side task)
```powershell
.\winpeas.exe
whoami /priv              # look for SeImpersonatePrivilege -> Potato exploits
wmic service get name,pathname | findstr /i /v "C:\Windows\\"   # unquoted paths
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

## Remediation
```powershell
Disable-LocalUser -Name <name>   # or net user <name> /active:no
Remove-LocalGroupMember -Group "Administrators" -Member <name>
Unregister-ScheduledTask -TaskName <name>
Stop-Service <name>; sc.exe delete <name>
Set-MpPreference -DisableRealtimeMonitoring $false   # re-enable Defender if disabled
```

**Full detail:** `windows/01..05-*.md` | **Rules reminder:** `00-rules-and-reminders.md`
