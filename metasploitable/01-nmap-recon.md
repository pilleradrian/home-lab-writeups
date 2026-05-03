# Reconnaissance — nmap network scan
**Target:** Metasploitable 2 — 10.0.2.3  
**Attacker:** Kali Linux — 10.0.2.5  
**Goal:** Discover open ports, running services, and software versions on the target machine

## Summary
The target is running Linux kernel 2.6.x with 23 open TCP ports. There is a lot to work with here, multiple services are old, badly configured, or have known backdoors baked in. Getting root on this machine is going to be straightforward through several different paths.

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

## Open ports and services
 
| Port | Service | Version | Risk |
|------|---------|---------|------|
| 21 | FTP | vsftpd 2.3.4 | Critical — known backdoor, anonymous login enabled |
| 22 | SSH | OpenSSH 4.7p1 | Medium — old version |
| 23 | Telnet | Linux telnetd | High — no encryption, passwords go over the wire in plaintext |
| 25 | SMTP | Postfix | Low — SSLv2 supported, certificate expired in 2010 |
| 53 | DNS | ISC BIND 9.4.2 | Medium — old version, zone transfer probably works |
| 80 | HTTP | Apache 2.2.8 | High — hosts several vulnerable web apps |
| 139 | SMB | Samba 3.0.20 | Critical — known RCE bug |
| 445 | SMB | Samba 3.0.20 | Critical — same as above |
| 512 | rexec | netkit-rsh | High — old unencrypted remote access protocol |
| 513 | rlogin | rlogind | High — same |
| 514 | rsh | tcpwrapped | High — same |
| 1099 | Java RMI | grmiregistry | Medium — could be hit with deserialization attacks |
| 1524 | bindshell | Metasploitable root shell | Critical — there is literally an open shell sitting here |
| 2049 | NFS | RPC | Medium — might expose filesystem shares |
| 2121 | FTP | ProFTPD 1.3.1 | High — has known exploits |
| 3306 | MySQL | 5.0.51a | High — open to the network, probably no root password |
| 5432 | PostgreSQL | 8.3.0 | High — open to the network, old version |
| 5900 | VNC | Protocol 3.3 | High — weak password, easy to brute force |
| 6000 | X11 | — | Medium — display server is exposed |
| 6667 | IRC | UnrealIRCd 3.2.8.1 | Critical — known backdoor |
| 8009 | AJP | Apache JServ 1.3 | High — Ghostcat vulnerability |
| 8180 | HTTP | Apache Tomcat 5.5 | High — probably still running default credentials |
 
---
 
## Critical findings
 
### 1. Open shell on port 1524
There is a root shell just sitting open on port 1524. No exploit needed — you connect and you're in.
 
**Command used:**
```
nc 10.0.2.3 1524
```
 
**What this means:** You get a root shell immediately. From there you can read any file on the system, add users, or use it as a jumping-off point to attack other machines.
 
**How to fix it:** Close port 1524 at the firewall. Regularly check what's listening on your machine with `netstat -tulnp` and kill anything that shouldn't be there.
 
---
 
### 2. vsftpd 2.3.4 backdoor — port 21 (CVE-2011-2523)
Someone snuck a backdoor into the vsftpd source code before it was published — a classic supply chain attack. Sending `:)` as the username triggers a root shell on port 6200.
 
**What this means:** Root access over FTP, remotely.
 
**How to fix it:** Upgrade vsftpd past version 2.3.4. Check the SHA256 checksum of anything you download before installing it. This is exactly why you don't grab random tarballs off the internet.
 
---
 
### 3. Samba 3.0.20 — port 445 (CVE-2007-2447)
The username map script in this version of Samba passes whatever you type straight to a shell command without checking it first. So you just type a command instead of a username and it runs as root.
 
**What this means:** Root access over SMB, remotely.
 
**How to fix it:** Update Samba. If you don't need file sharing, just turn it off. Either way, don't let SMB talk to the internet.
 
---
 
### 4. UnrealIRCd 3.2.8.1 backdoor — port 6667 (CVE-2010-2075)
Same story as vsftpd — someone put a backdoor in the source code. Sending a specific sequence of bytes makes the server run whatever command you send.
 
**What this means:** Remote code execution through an IRC server.
 
**How to fix it:** Update UnrealIRCd. Verify checksums before compiling anything from source.
 
---
 
## High risk findings
 
### 5. MySQL open to the network — port 3306
MySQL is listening on all interfaces instead of just localhost, and the root account probably has no password.
 
**Try this:**
```
mysql -h 10.0.2.3 -u root
```
 
**How to fix it:** Add `bind-address = 127.0.0.1` to my.cnf so MySQL only listens locally. Put a password on every account.
 
---
 
### 6. Apache Tomcat 5.5 — port 8180
Old Tomcat, almost certainly still running the default credentials. If you get into the manager panel you can upload a WAR file and get a shell.
 
**Try:** `http://10.0.2.3:8180/manager` with `tomcat/tomcat` or `admin/admin`
 
**How to fix it:** Change the default credentials on day one. Remove the manager app if you don't need it. Update Tomcat.
 
---
 
### 7. Telnet — port 23
Telnet sends everything over the wire with no encryption. If you log in over Telnet and someone is running Wireshark on the same network, they see your password.
 
**How to fix it:** Turn off Telnet. Use SSH.
 
---
 
### 8. Anonymous FTP login — port 21
You can log into the FTP server with no credentials at all. Username `anonymous`, any password, and you're in.
 
**How to fix it:** Turn off anonymous login unless you actually need it. If you do need it, give anonymous users their own directory with nothing sensitive in it.
 
---
 
### 9. rexec, rlogin, rsh — ports 512, 513, 514
These three protocols are from the 1980s and have basically no security. No encryption, minimal authentication. They have no place on a modern system.
 
**How to fix it:** Disable all three. Use SSH instead.
 
---
 
### 10. VNC with a weak password — port 5900
VNC is reachable from the network and is using protocol version 3.3 from 1998. The password is weak enough to brute force in a few minutes.
 
**How to fix it:** Set a strong VNC password. Don't expose VNC directly — tunnel it through SSH if you need remote desktop access.
 
---
 
## Other things worth noting
 
- **Expired SSL cert on SMTP:** The certificate expired in 2010 and the server still accepts SSLv2, which has been broken for decades.
- **DNS zone transfer (port 53):** BIND 9.4.2 will probably hand over the entire DNS zone to anyone who asks. This leaks internal hostnames and IPs.
- **NFS (port 2049):** Depending on how exports are configured, you might be able to mount the filesystem remotely without any authentication.
- **SMB signing disabled:** Means SMB traffic can be intercepted and relayed to authenticate as another user.
## What to attack first
 
| Priority | Port | How | What you get |
|----------|------|-----|--------------|
| 1 | 1524 | netcat | Instant root shell |
| 2 | 21 | vsftpd Metasploit module | Root shell |
| 3 | 445 | Samba Metasploit module | Root shell |
| 4 | 3306 | mysql client, no password | Full database access |
| 5 | 8180 | Default Tomcat creds + WAR file | Shell via web |
| 6 | 6667 | UnrealIRCd Metasploit module | Remote code execution 

## What I actually took away from this
 
- You only need one thing to go wrong to lose the whole machine. This one has over a dozen.
- The vsftpd and UnrealIRCd backdoors are a good reminder that the threat doesn't always come from outside,sometimes the software itself is already compromised when you install it.
- Every service running on a machine is something else that can be attacked. If you don't need it, turn it off.
---
## Next steps
- [ ] Connect to port 1524 with netcat
- [ ] Exploit vsftpd via Metasploit
- [ ] Exploit Samba via Metasploit
- [ ] Try MySQL with no password
- [ ] Try Tomcat default credentials
- [ ] Exploit UnrealIRCd via Metasploit
- [ ] Capture Telnet credentials with Wireshark
- [ ] Check NFS exports
- [ ] Attempt DNS zone transfer
