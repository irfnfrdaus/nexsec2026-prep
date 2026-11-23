# Windows — Active Directory Enumeration

If an assigned Windows host turns out to be domain-joined (or is a domain controller itself), the
checks in `01-04` still apply, but add these AD-specific ones.

## 1. Confirm domain membership and role

```powershell
systeminfo | findstr /i "domain"
Get-ADDomain 2>$null   # if RSAT/AD module is available
nltest /dsgetdc:<domain>   # find the domain controller
whoami /fqdn
```

## 2. Enumerate users, groups, and privileged accounts

```powershell
Get-ADUser -Filter * -Properties LastLogonDate, PasswordLastSet | Select Name, LastLogonDate, PasswordLastSet
Get-ADGroupMember -Identity "Domain Admins"
Get-ADGroupMember -Identity "Enterprise Admins" 2>$null
net group "Domain Admins" /domain      # works without the AD PowerShell module
net group "Domain Controllers" /domain
```

Without RSAT tools available, `net user`, `net group`, and `nltest` still work from any domain-joined
box and don't require special setup.

## 3. Look for common AD misconfigurations

```powershell
# Kerberoastable accounts — service accounts with SPNs set, crackable offline if you can request a TGS
setspn -T <domain> -Q */*

# AS-REP roastable accounts — accounts with Kerberos pre-auth disabled
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth 2>$null

# Unconstrained/constrained delegation — high-value privesc/lateral-movement target if misconfigured
Get-ADComputer -Filter {TrustedForDelegation -eq $true} 2>$null
```

If you don't have the AD module, tools like **PowerView** or **BloodHound/SharpHound** (if permitted
and available in the environment) automate this enumeration and visualize attack paths — check with
the organizers whether such tools are appropriate/available before relying on them, since they can be
noisy and the environment may be shared/limited.

## 4. Group Policy review

```powershell
Get-GPO -All 2>$null
gpresult /h gpreport.html   # generates a readable report of applied policy on the current machine
```

Look for GPOs that weaken security (disabled password complexity, disabled lockout policy, overly
permissive logon rights) — these can be both the entry point and something to remediate.

## 5. Trust relationships (if relevant)

```powershell
nltest /domain_trusts
```

## 6. Event log IDs specific to AD activity

| ID | Meaning |
|---|---|
| 4768 | Kerberos TGT requested |
| 4769 | Kerberos service ticket requested (look for unusual/bulk requests — Kerberoasting indicator) |
| 4776 | Domain controller attempted to validate credentials (NTLM) |
| 5136 | Directory service object modified |

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4769} -MaxEvents 100 |
  Group-Object { $_.Properties[1].Value } | Sort Count -Descending   # bulk TGS requests from one account = possible Kerberoasting
```

## Remediation notes specific to AD

- [ ] Reset the `krbtgt` account password **twice** if you suspect a Golden Ticket attack (only if
      genuinely indicated and understood — this is disruptive and should be a deliberate team decision,
      not a reflex).
- [ ] Remove unauthorized accounts from privileged groups (Domain Admins, Enterprise Admins).
- [ ] Disable Kerberos pre-auth exceptions and unnecessary SPNs where found.
- [ ] Tighten delegation settings found to be unconstrained without a clear business reason.
- [ ] Re-confirm required services (authentication, file shares, etc.) still work after any AD change —
      AD changes have a much higher blast radius than a single host, so test carefully.
