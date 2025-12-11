

DC ports:
```
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 126 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 126 Microsoft Windows Kerberos (server time: 2025-11-20 15:54:20Z)
135/tcp  open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 126 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 126 Microsoft Windows Active Directory LDAP (Domain: northbridge.corp0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=NORTHDC01.northbridge.corp
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:NORTHDC01.northbridge.corp
| Issuer: commonName=northbridge-NORTHDC01-CA/domainComponent=northbridge
445/tcp  open  microsoft-ds? syn-ack ttl 126
464/tcp  open  kpasswd5?     syn-ack ttl 126
593/tcp  open  ncacn_http    syn-ack ttl 126 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 126 Microsoft Windows Active Directory LDAP (Domain: northbridge.corp0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=NORTHDC01.northbridge.corp
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:NORTHDC01.northbridge.corp
| Issuer: commonName=northbridge-NORTHDC01-CA/domainComponent=northbridge
3268/tcp open  ldap          syn-ack ttl 126 Microsoft Windows Active Directory LDAP (Domain: northbridge.corp0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=NORTHDC01.northbridge.corp
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:NORTHDC01.northbridge.corp
| Issuer: commonName=northbridge-NORTHDC01-CA/domainComponent=northbridge
3269/tcp open  ssl/ldap      syn-ack ttl 126 Microsoft Windows Active Directory LDAP (Domain: northbridge.corp0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=NORTHDC01.northbridge.corp
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:NORTHDC01.northbridge.corp
| Issuer: commonName=northbridge-NORTHDC01-CA/domainComponent=northbridge
3389/tcp open  ms-wbt-server syn-ack ttl 126 Microsoft Terminal Services
| ssl-cert: Subject: commonName=NORTHDC01.northbridge.corp
| Issuer: commonName=NORTHDC01.northbridge.corp
|   Target_Name: NORTHBRIDGE
|   NetBIOS_Domain_Name: NORTHBRIDGE
|   NetBIOS_Computer_Name: NORTHDC01
|   DNS_Domain_Name: northbridge.corp
|   DNS_Computer_Name: NORTHDC01.northbridge.corp

```

NORTHJMP01
```
PORT     STATE SERVICE       REASON          VERSION
135/tcp  open  msrpc         syn-ack ttl 126 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 126 Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds? syn-ack ttl 126
3389/tcp open  ms-wbt-server syn-ack ttl 126 Microsoft Terminal Services
|_ssl-date: 2025-11-20T16:04:36+00:00; -2h00m12s from scanner time.
| ssl-cert: Subject: commonName=NORTHJMP01.northbridge.corp
| Issuer: commonName=NORTHJMP01.northbridge.corp

```

---

