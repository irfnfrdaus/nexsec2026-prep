# Offensive — Linux Privilege Escalation (General CTF / HTB-style)

## Step 1: run an automated enumeration script first

```bash
# LinPEAS — the standard first move, flags likely privesc vectors automatically
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh | tee linpeas_output.txt
```

Read the output top to bottom — it color-codes likely wins (red/yellow). Don't skip straight to the
bottom; sometimes the easiest vector (a cron job, a writable file) is flagged early.

## Then confirm manually — the classic checklist

### sudo rights
```bash
sudo -l
```
Cross-reference any binary listed against **GTFOBins** (gtfobins.github.io) — most common binaries
(`vim`, `find`, `less`, `awk`, `python`, `tar`, `cp`, etc.) with sudo rights have a documented
shell-escape technique.

### SUID binaries
```bash
find / -perm -4000 -type f 2>/dev/null
```
Same idea — check each result against GTFOBins for a SUID-specific escalation path.

### Cron jobs running as root
```bash
cat /etc/crontab; ls /etc/cron.d/
```
If a root cron job runs a script that's writable by your current user, edit it to add a reverse
shell or privilege-granting command.

### Writable `/etc/passwd`
```bash
ls -la /etc/passwd
```
If writable, you can append a new root user with a known password hash
(`openssl passwd -1 password123` to generate one) directly into `/etc/passwd`.

### NFS no_root_squash
```bash
cat /etc/exports   # on the NFS server, if accessible
```
A share exported with `no_root_squash` lets a remote root user create a SUID binary on the share that
executes as root on the target.

### Kernel exploits (last resort — can crash the box)
```bash
uname -a
# Cross-reference kernel version against known privesc CVEs (e.g. DirtyCow, PwnKit/CVE-2021-4034,
# Dirty Pipe/CVE-2022-0847) — only use in a CTF context, never casually in the real exercise since the
# guideline explicitly prohibits actions that permanently damage exercise infrastructure.
```

### Capabilities (often missed)
```bash
getcap -r / 2>/dev/null
```
A binary with `cap_setuid+ep` or similar can often be abused for escalation — check GTFOBins'
capabilities section too.

### Password reuse / credentials lying around
```bash
grep -r "password" /etc /opt /var/www 2>/dev/null | grep -v Binary
find / -name "*.bak" -o -name "*.old" -o -name "id_rsa*" 2>/dev/null
cat ~/.bash_history
```

## Workflow

1. Run LinPEAS, read the output.
2. Check `sudo -l` and SUID list against GTFOBins — this alone solves the majority of CTF-style privesc.
3. Check cron, NFS, writable config files for root-owned automation you can hijack.
4. Only reach for a kernel exploit if nothing else works, and understand it risks crashing the box.
