# VAPT Writeup #5 — TryHackMe: Vulnversity

**Exploiting an Unrestricted File Upload and SUID `systemctl` Misconfiguration**

| | |
|---|---|
| **Author** | Ashwin Baburaj |
| **Date** | 19–20 August 2026 |
| **Lab Environment** | TryHackMe — Vulnversity room, accessed via personal Kali VM over OpenVPN |
| **Attacker Machine** | Kali Linux (VPN IP: `192.168.163.234`) |
| **Target Machine** | THM-hosted Vulnversity box (`10.48.163.67`) |
| **Tools Used** | Nmap, Gobuster, Burp Suite Community (Proxy + Intruder), Netcat, PHP reverse shell (Pentestmonkey) |
| **Vulnerability Class** | Unrestricted file upload (blacklist bypass) + SUID misconfiguration on `/bin/systemctl` |

---

## 1. Scope & Objective

This engagement was carried out against the Vulnversity room on TryHackMe, connected via a personal Kali Linux VM over the platform's OpenVPN service (rather than the browser-based AttackBox), to get hands-on practice with a realistic attacker workflow end to end. No production systems were involved. The objective was to identify a web application vulnerability, gain an initial foothold, and escalate to full root access through the standard VAPT process: reconnaissance, enumeration, exploitation, post-exploitation, and privilege escalation.

## 2. Reconnaissance

A full TCP port scan with service/version detection and safe-script scanning was run against the target. An initial default-timing scan under-reported open ports due to minor packet loss over the VPN link, so the scan was re-run with adjusted timing and retry settings:

```bash
nmap -sC -sV -p- --min-rate=300 --max-retries=3 -Pn 10.48.163.67 -oN nmap_full.txt
```

![Nmap scan results](readme_images/nmap.png)
*Figure 1 — Nmap scan revealing FTP, SSH, Samba, Squid proxy, and an Apache instance titled "Vuln University" on port 3333.*

The scan identified six open services, most notably:

- Port 21 — vsftpd 3.0.5
- Port 22 — OpenSSH 8.2p1
- Ports 139/445 — Samba smbd 4
- Port 3128 — Squid http-proxy 4.10
- **Port 3333 — Apache httpd 2.4.41, page titled "Vuln University"**

Port 3333 stood out as the primary attack surface — a custom web application rather than a default service page, making it the logical next target for enumeration.

## 3. Enumeration

Directory brute-forcing was run against the web application on port 3333 to discover content not linked from the homepage navigation:

```bash
gobuster dir -u http://10.48.163.67:3333 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40
```

![Gobuster results](readme_images/gobuster.png)
*Figure 2 — Gobuster uncovers a hidden `/internal/` directory (HTTP 301) alongside standard asset folders.*

The `/internal/` path resolved to a file upload form with no authentication in front of it — a strong candidate for an unrestricted file upload vulnerability.

## 4. Exploitation — Bypassing the Upload Filter

A benign `test.php` file was uploaded through the form while intercepting the request in Burp Suite's Proxy. The server rejected the `.php` extension outright ("Extension not allowed"), confirming an extension blacklist was in place:

![Burp intercept](readme_images/burp_intercept.png)
*Figure 3 — Burp Proxy intercepting the multipart/form-data POST request carrying the uploaded filename.*

The intercepted request was sent to Burp Intruder to fuzz the file extension against a small custom wordlist of common PHP-executable variants (`php3`, `php4`, `php5`, `php7`, `pht`, `phtml`, `phar`), with the payload position set on the extension only:

![Intruder payload position](readme_images/intruder_position.png)
*Figure 4 — Payload position isolated to the file extension (`test.§php§`) ahead of the Intruder attack.*

Most extensions returned an identical rejection response. One entry stood apart — **`phtml`**


## 6. Privilege Escalation — SUID `systemctl`

With a foothold established, the filesystem was searched for SUID binaries — files that execute with their owner's privileges regardless of who runs them, a classic Linux privesc vector:

```bash
find / -perm -u=s -type f 2>/dev/null
```

Among the results, **`/bin/systemctl`** stood out immediately — this binary should never carry the SUID bit on a properly hardened system. Its permissions were confirmed directly:

```bash
ls -la /bin/systemctl
```

![SUID confirmation](readme_images/suid_confirm.png)
*Figure 7 — `/bin/systemctl` owned by root with the SUID bit set (`-rwsr-xr-x`), confirming a viable privilege escalation path.*

This is a documented [GTFOBins](https://gtfobins.github.io/gtfobins/systemctl/) technique: `systemctl status` pipes long output through the `less` pager. Escaping `less` with a shell command while `systemctl` is running as root (via SUID) spawns that shell with the same elevated privileges:

```bash
systemctl status
# once paused inside the pager:
!/bin/sh -p
```

The `-p` flag was essential — it instructs the spawned shell to preserve its effective privileges rather than dropping back to the real (unprivileged) UID, which is the default protective behaviour of most shells when invoked with mismatched real/effective IDs.

This escape dropped into a root shell on the target. Navigating to `/root` and reading the flag confirmed full compromise:

```bash
cd /root
ls
cat root.txt
```

![Root flag](readme_images/root_flag.png)
*Figure 8 — Root shell confirmed (`root@vulnuniversity`) with the root flag read from `/root/root.txt`.*

**Root flag captured:**


## 7. Impact

Chained together, these two flaws allow a fully unauthenticated remote attacker to:

- Upload and execute arbitrary PHP code on the target purely by bypassing a client-facing extension blacklist
- Obtain an initial foothold as the web service account with no credentials required
- Escalate that foothold directly to root using a single misconfigured SUID binary already present on the system
- Gain complete, unrestricted control of the host — read/write any file, install persistence, or pivot further into the network

## 8. Remediation

- Validate uploads server-side by actual file content/MIME type, not just extension — reject anything that isn't an expected type regardless of blacklist coverage
- Store uploaded files outside the webroot, or disable script execution in the upload directory, so even a successfully uploaded script cannot run
- Never assign the SUID bit to `systemctl`, or any binary capable of spawning a shell or pager — audit SUID binaries regularly with `find / -perm -u=s -type f`
- Apply the principle of least privilege to the web service account so that even a full web-app compromise doesn't expose a path to local privilege escalation
- Keep systemd and other core system utilities on their vendor-default permissions; treat any deviation from a standard package's file permissions as a red flag during a hardening review

## 9. Conclusion

This engagement demonstrated a complete, realistic attack chain — from a hidden upload endpoint discovered through directory brute-forcing, to a filter bypass identified via targeted Burp Intruder fuzzing, to a textbook SUID privilege escalation that handed over full root access. Unlike writeup #1's single-step compromise, this exercise required chaining two distinct, independent misconfigurations, reinforcing the value of methodical enumeration at every stage and the outsized impact a single overlooked file permission (SUID on `systemctl`) can have even after a foothold is already limited to a low-privileged account.
