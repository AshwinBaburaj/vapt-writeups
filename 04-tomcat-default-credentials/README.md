# [Target/Service Name] — [Vulnerability Name]

**CVE:** [CVE-XXXX-XXXXX or "N/A — Misconfiguration"]
**Target:** [Metasploitable2 / TryHackMe room name]
**Date:** [Month Year]
**Severity:** [Critical / High / Medium]

## Summary

One or two sentences: what the vulnerability is and what access it grants.

## 1. Reconnaissance

- Nmap scan command + relevant output (ports, services, versions discovered)
- What led you to suspect this vulnerability

```
$ nmap -sV -p- <target-ip>
[paste relevant output]
```

## 2. Vulnerability Identification

- How you confirmed the vulnerability (version match, manual check, searchsploit, etc.)
- Reference to CVE / advisory / exploit-db entry

## 3. Exploitation

- Step-by-step exploitation process
- Tool/exploit used (Metasploit module, manual exploit, custom script)
- Screenshot(s) of successful exploitation / shell access

```
[commands used]
```

## 4. Impact

- What level of access was achieved (root / user / data exposure)
- What an attacker could do with this access in a real-world scenario

## 5. Remediation

- How this vulnerability should be fixed/patched
- Version to upgrade to, config change, or compensating control

## References

- [CVE link]
- [Exploit-DB / advisory link]
