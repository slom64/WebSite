[[RustyKey]]

# Enumeration
```sh
nxc smb -M timeroast
# REMOVE THE TRAILING CHARACTERS BEFORE $ sign
# 1000:$sntp-ms$09a7c732a9f93fdc46310f2689da4c62$1c0111e900000000000a7d174c4f434cecb5bb0fd93fd786e1b8428bffbfcd0aecb65ff9fd16cbf4ecb65ff9fd16ef2f
# $sntp-ms$d93c7a6c3eb840a99b18fe92cd5883e3$1c0111e900000000000a7d174c4f434cecb5bb0fd8b9fc0fe1b8428bffbfcd0aecb65ffab8b9dc2fecb65ffab8ba0b29
./hashcat.bin -m 31300 -a 0 ~/HTB/windows/hard/RustyKey/timeRoasting_hash.txt  /opt/lists/rockyou.txt
```

# When to do it?
If you are able to see informations about computers in domain, check when was the last time did someone changed the computers password. If it wasn't too far away, it worth check.


# Conditions
1. The target must be a computer account; ordinary user accounts cannot be directly targeted (unless "Target Timeroasting" is used to modify properties). 
2. The target domain controller must have the NTP service running with Microsoft SNTP Extended Authentication (MS-SNTP) enabled and responding, with UDP port 123 open. 
3. The attacker can send unauthenticated MS-SNTP requests to the DC (without valid credentials). 
4. The attacker can enumerate the RIDs (Relative Identifiers) of computer accounts in the domain. 
5. (Optional) For "Target Timeroasting," domain administrator privileges are required to temporarily modify user account properties to treat them as computer accounts. 6. The computer account passwords in the domain are not strongly protected (e.g., weak passwords or not regularly changed).

---

Short answer: **Timeroasting is not the classic Kerberos (service-ticket) attack — it’s a different “roasting” technique that abuses Windows’ time/NTP handling to obtain password-equivalent hashes (usually of computer accounts) for offline cracking.** It’s conceptually similar to Kerberoasting/AS-REP roasting (goal = get material you can brute-force offline) but it targets the Windows Time / MS-SNTP mechanism rather than Kerberos service tickets. ([Bureau Veritas Cybersecurity](https://cybersecurity.bureauveritas.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf?utm_source=chatgpt.com "TIMEROASTING, TRUSTROASTING AND COMPUTER ..."))

## What Timeroasting is (high level)
- Windows implements an MS-SNTP extension for the Windows Time service. That protocol can return a response that, if queried in a particular way, contains a value that is effectively a **password-equivalent hash** for a machine account (derived from the account’s secret). An unauthenticated requester can trigger those responses for specific account RIDs and harvest those hashes. Attackers then try to crack them offline. ([Bureau Veritas Cybersecurity](https://cybersecurity.bureauveritas.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf?utm_source=chatgpt.com "TIMEROASTING, TRUSTROASTING AND COMPUTER ..."))
## How it compares to Kerberoasting / AS-REP roasting (conceptually)
- **Kerberoasting:** request Kerberos service tickets for SPNs → tickets are encrypted with the service account’s key → extract ticket and crack offline to get service account password. ([CrowdStrike](https://www.crowdstrike.com/en-us/cybersecurity-101/cyberattacks/kerberoasting/?utm_source=chatgpt.com "What is a Kerberoasting Attack?"))
- **AS-REP roasting:** target accounts that don’t require pre-authentication (AS-REP contains material that can be brute-forced offline). ([Medium](https://timurengin.com/kerberos-as-rep-roasting-kerberoasting-anatomy-prevention-and-detection-84d6634db2af?utm_source=chatgpt.com "Kerberos AS-REP Roasting (Kerberoasting) - Timur Engin"))
- **Timeroasting:** target Windows Time / MS-SNTP responses to obtain hashes of **computer** (and with certain configurations, other) accounts without needing domain creds. It’s unauthenticated and yields offline-crackable data. ([Swissky's Lab](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-roasting-timeroasting/?utm_source=chatgpt.com "Roasting - Timeroasting - Internal All The Things"))

## Why practitioners care
- It’s **low-cost** (no creds needed) and targets often-ignored computer account passwords (which may be weak or machine-managed in ways that make attack feasible). It expands the set of accounts an attacker can roast beyond just service accounts. ([Bureau Veritas Cybersecurity](https://cybersecurity.bureauveritas.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf?utm_source=chatgpt.com "TIMEROASTING, TRUSTROASTING AND COMPUTER ..."))
## High-level mitigations / detection (don’t need commands here)
- Patch Windows Time / apply vendor fixes and recommended hardening from Microsoft/security vendors. ([Bureau Veritas Cybersecurity](https://cybersecurity.bureauveritas.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf?utm_source=chatgpt.com "TIMEROASTING, TRUSTROASTING AND COMPUTER ..."))
- Reduce attack surface: restrict which hosts can respond to NTP/MS-SNTP queries, firewall NTP traffic, and limit unauthenticated NTP where possible. ([Swissky's Lab](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-roasting-timeroasting/?utm_source=chatgpt.com "Roasting - Timeroasting - Internal All The Things"))
- Ensure strong, non-guessable computer account passwords / rotation and avoid weak machine password practices. ([Bureau Veritas Cybersecurity](https://cybersecurity.bureauveritas.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf?utm_source=chatgpt.com "TIMEROASTING, TRUSTROASTING AND COMPUTER ..."))
- Monitor/log unusual NTP activity and look for mass/targeted NTP requests that map to many RIDs; add alerts for large numbers of MS-SNTP queries from odd sources. ([Swissky's Lab](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-roasting-timeroasting/?utm_source=chatgpt.com "Roasting - Timeroasting - Internal All The Things"))

---
If you want, I can:
- give a concise detection checklist you can use on an engagement (event IDs / logs to look for, network signs) **or**
- summarize the Secura whitepaper (concise points only), or
- show a short comparison table of Timeroast vs Kerberoast vs AS-REP roast.
Which of those helps you next?