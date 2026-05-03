# Reconnaissance — nmap network scan
**Target:** Metasploitable 2 — 10.0.2.3  
**Attacker:** Kali Linux — 10.0.2.5  
**Goal:** Discover open ports, running services, and software versions on the target machine

---

## Commands run

### Host discovery
```
nmap -sn 10.0.2.0/24
```

### Port and service scan
```
nmap -sV -p- 10.0.2.3
```

### Aggressive scan (OS + version + scripts)
```
nmap -A -T4 10.0.2.3
```

---

## Output

### Host discovery results
```
See 
```

### Service scan results
```
[paste output here]
```

---

## Findings

| Port | Protocol | Service | Version | Notes |
|------|----------|---------|---------|-------|
| 21 | TCP | FTP | vsftpd 2.3.4 | Known backdoor CVE-2011-2523 |
| 22 | TCP | SSH | OpenSSH 4.7p1 | Old version |
| 80 | TCP | HTTP | Apache 2.2.8 | Web server running |
| 139 | TCP | SMB | Samba 3.x | CVE-2007-2447 |
| 3306 | TCP | MySQL | 5.0.51a | No auth on some versions |

*Add or remove rows based on your actual scan results.*

---

## Interesting services

### [Service name — e.g. vsftpd 2.3.4 on port 21]
- **Why interesting:** Explain why this caught your attention
- **CVE:** CVE-XXXX-XXXX
- **Potential impact:** What an attacker could do if this is exploited

### [Service name — e.g. Samba on port 139]
- **Why interesting:**
- **CVE:**
- **Potential impact:**

*Add a section for each service worth investigating.*

---

## What I learned
- What surprised you about the scan results?
- What did you not expect to find?
- What would you look at first if this were a real engagement and why?

---

## Next steps
- [ ] Look up CVEs for each interesting service on exploit-db.com
- [ ] Try exploiting vsftpd 2.3.4 backdoor
- [ ] Try exploiting Samba usermap vulnerability
- [ ] Check the web server on port 80 for further attack surface

---

## Remediation recommendations
- **vsftpd:** Upgrade to a version after 2.3.4, verify checksums on downloaded software
- **Samba:** Patch to latest version or disable if not needed, restrict to internal network
- **SSH:** Upgrade to latest OpenSSH, disable password auth, use key-based auth only
- **MySQL:** Require authentication, restrict to localhost if no remote access needed
- **General:** Disable or remove any service that is not actively needed (principle of least exposure)
