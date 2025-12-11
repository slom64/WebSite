summary



---

IP: 10.10.11.87

We will start with nmap:
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 10.0p2 Debian 8 (protocol 2.0)     
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel 
```

Found this ssh is vuln to several CVE: 
- CVE-2025-32728 `can't find `, 
- CVE-2025-26465 `MiMT` 
- CVE-2025-26466
	- It says that debain uses older versions of ssh even if it has the newer.
- CVE-2024-6387 `old`
None has worked


---

Do `UDP` scan and found port 500 is open.
```
500/udp open  isakmp
```

asking chatgpt found that this tool can give us the key that is used for ssh. it gave it us back as hased and we crack it

`ike` : `freakingrockstarontheroad`

---
