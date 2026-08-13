# VAPT Home Lab — Writeups

Structured penetration testing writeups from an isolated home lab (VirtualBox: Kali Linux attacker VM, target VMs on a shared NAT network). Each writeup follows a consistent format: **Recon → Identification → Exploitation → Impact → Remediation**.

## Writeups

| # | Target | Vulnerability | CVE | Status |
|---|--------|---------------|-----|--------|
| 01 | Metasploitable2 | vsftpd 2.3.4 Backdoor | [CVE-2011-2523](01-vsftpd-cve-2011-2523/) | ✅ Complete |
| 02 | Metasploitable2 | UnrealIRCd Backdoor | [CVE-2010-2075](02-unrealircd-cve-2010-2075/) | ✅ Complete |
| 03 | Metasploitable2 | Samba Anonymous Access | [Writeup](03-samba-anonymous-access/) | ✅ Complete |
| 04 | Metasploitable2 | Tomcat Default Credentials | [Writeup](04-tomcat-default-credentials/) | ✅ Complete |
| 05 | TryHackMe — Vulnversity | Web App File Upload → RCE | [Writeup](05-vulnversity/) | 🔜 In progress |
| 06 | TryHackMe — Kenobi | Samba/rsync Misconfig → PATH Hijack Privesc | [Writeup](06-kenobi/) | 🔜 In progress |

## Lab Environment

- **Attacker:** Kali Linux (VirtualBox)
- **Targets:** Metasploitable2 (writeups 1–4), TryHackMe rooms (writeups 5–6)
- **Network:** Shared NAT Network (VirtualBox)
- **Tools:** Nmap, Metasploit Framework, Burp Suite, LinPEAS

## About

Built as part of a structured, self-directed skill-building plan to develop hands-on VAPT experience alongside a cybersecurity internship at Maharashtra State Cyber Headquarters.
