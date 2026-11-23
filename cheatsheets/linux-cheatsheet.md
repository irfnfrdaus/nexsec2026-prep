# Linux Cheat Sheet (print this one page)

## Triage
```
hostname; uname -a; whoami; id; uptime
w; last -a | head -30; lastlog
ps auxww --forest
ss -tulpn ; ss -antp | grep ESTAB
df -h
```

## Accounts
```
awk -F: '$3 == 0 {print}' /etc/passwd     # all UID 0 accounts
cat /etc/sudoers; ls /etc/sudoers.d/
cat /etc/shadow | awk -F: '{print $1,$2}'  # lock status
```

## Persistence (check ALL of these)
```
for home in /root /home/*; do cat "$home/.ssh/authorized_keys" 2>/dev/null; done
crontab -l -u root; cat /etc/crontab; ls /etc/cron.*/
systemctl list-units --type=service --all | grep -v 'loaded active'
find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null
cat /etc/ld.so.preload
for f in /etc/profile /root/.bashrc; do cat "$f"; done
```

## Logs
```
grep "Failed password" /var/log/auth.log | tail -50
grep "Accepted" /var/log/auth.log
grep "sudo:" /var/log/auth.log
journalctl -u ssh --since "2 days ago"
journalctl _COMM=sudo --since today
```

## Privesc (if offensive-side task)
```
curl -L .../linpeas.sh -o linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh
sudo -l
find / -perm -4000 -type f 2>/dev/null   # cross-ref GTFOBins
getcap -r / 2>/dev/null
```

## Remediation
```
usermod -L <user> ; passwd -l <user>
systemctl restart sshd   # after sshd_config changes — test new session BEFORE closing current!
ss -tulpn   # re-check after cleanup
```

**Full detail:** `linux/01..04-*.md` | **Rules reminder:** `00-rules-and-reminders.md`
