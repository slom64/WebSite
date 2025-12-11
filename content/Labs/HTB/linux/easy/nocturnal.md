amanda ---> arHkG7HAI68X8s1J
tobias  ---> slowmotionapocalypse

```php
function cleanEntry($entry) {
    $blacklist_chars = [';', '&', '|', '$', ' ', '`', '{', '}', '&&'];

    foreach ($blacklist_chars as $char) {
        if (strpos($entry, $char) !== false) {
            return false; // Malicious input detected
        }
    }

    return htmlspecialchars($entry, ENT_QUOTES, 'UTF-8');
}


if (isset($_POST['backup']) && !empty($_POST['password'])) {
    $password = cleanEntry($_POST['password']);
    $backupFile = "backups/backup_" . date('Y-m-d') . ".zip";

    if ($password === false) {
        echo "<div class='error-message'>Error: Try another password.</div>";
    } else {
        $logFile = '/tmp/backup_' . uniqid() . '.log';
       
        $command = "zip -x './backups/*' -r -P " . $password . " " . $backupFile . " .  > " . $logFile . " 2>&1 &";
        
        $descriptor_spec = [
            0 => ["pipe", "r"], // stdin
            1 => ["file", $logFile, "w"], // stdout
            2 => ["file", $logFile, "w"], // stderr
        ];

        $process = proc_open($command, $descriptor_spec, $pipes);
        if (is_resource($process)) {
            proc_close($process);
        }

        sleep(2);

        $logContents = file_get_contents($logFile);
        if (strpos($logContents, 'zip error') === false) {
            echo "<div class='backup-success'>";
            echo "<p>Backup created successfully.</p>";
            echo "<a href='" . htmlspecialchars($backupFile) . "' class='download-button' download>Download Backup</a>";
            echo "<h3>Output:</h3><pre>" . htmlspecialchars($logContents) . "</pre>";
            echo "</div>";
        } else {
            echo "<div class='error-message'>Error creating the backup.</div>";
        }

        unlink($logFile);
    }
}


```

```
'admin','d725aeba143f575736b07e045d8ceebb');
'amanda','df8b20aa0c935023f99ea58358fb63c4');
'tobias','55c82b1ccd55ab219b3b109b07d5061d'); --> slowmotionapocalypse
'kavi','f38cde1654b39fea2bd4f72f1ae4cdda');   --> kavi
'e0Al5','101ad4543a96a7fd84908fd0d802e7db');

```

```sh
netstat -lntp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:587           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      - 
```

```sh
nmap 

PORT      STATE  SERVICE    REASON         VERSION
25/tcp    closed smtp       reset ttl 64
53/tcp    closed domain     reset ttl 64
80/tcp    closed http       reset ttl 64
587/tcp   closed submission reset ttl 64
3306/tcp  closed mysql      reset ttl 64
8080/tcp  open   http-proxy syn-ack ttl 64 Burp Suite Professional http proxy
|_http-favicon: Unknown favicon MD5: A675F036BD20CD86921FAD84C29A8D04
|_http-title: Burp Suite Professional
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
33060/tcp closed mysqlx     reset ttl 64

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 08:15
Completed NSE at 08:15, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 08:15
Completed NSE at 08:15, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 08:15
Completed NSE at 08:15, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.40 seconds
           Raw packets sent: 7 (308B) | Rcvd: 15 (632B)

```

# checking
## mail
```sh
/var/backups
```

## databases
```sh
/etc/mysql/mysql.conf.d/mysqld.cnf
```


## wierd
```
╔══════════╣ Users with console                                                                                                                                                                    
ispapps:x:1001:1002::/var/www/apps:/bin/sh                                                                                                                                                         
ispconfig:x:1002:1003::/usr/local/ispconfig:/bin/sh                                                                                                                                                
root:x:0:0:root:/root:/bin/bash                                                                                                                                                                    
tobias:x:1000:1000:tobias:/home/tobias:/bin/bash
```

## cron.job
```sh
╔══════════╣ Check for vulnerable cron jobs                                                                                                                                                        
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                                                                               
══╣ Cron jobs list                                                                                                                                                                                 
/usr/bin/crontab                                                                                                                                                                                   
incrontab Not Found                                                                                                                                                                                
-rw-r--r-- 1 root root    1042 Feb 13  2020 /etc/crontab                                                                                                                                           
                                                                                                                                                                                                   
/etc/cron.d:                                                                                                                                                                                       
total 28                                                                                                                                                                                           
drwxr-xr-x   2 root root 4096 Oct 18  2024 .                                                                                                                                                       
drwxr-xr-x 110 root root 4096 Apr 17 08:54 ..                                                                                                                                                      
-rw-r--r--   1 root root  201 Feb 14  2020 e2scrub_all                                                                                                                                             
-rw-r--r--   1 root root  712 Mar 27  2020 php                                                                                                                                                     
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder                                                                                                    ╔══════════╣ Check for vulnerable cron jobs                                                                                                                                                        
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                                                                               
══╣ Cron jobs list                                                                                                                                                                                 
/usr/bin/crontab                                                                                                                                                                                   
incrontab Not Found                                                                                                                                                                                
-rw-r--r-- 1 root root    1042 Feb 13  2020 /etc/crontab                                                                                                                                           
                                                                                                                                                                                                   
/etc/cron.d:                                                                                                                                                                                       
total 28                                                                                                                                                                                           
drwxr-xr-x   2 root root 4096 Oct 18  2024 .                                                                                                                                                       
drwxr-xr-x 110 root root 4096 Apr 17 08:54 ..                                                                                                                                                      
-rw-r--r--   1 root root  201 Feb 14  2020 e2scrub_all                                                                                                                                             
-rw-r--r--   1 root root  712 Mar 27  2020 php                                                                                                                                                     
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder                                                                                                                                            
-rw-r--r--   1 root root  190 Mar 14  2023 popularity-contest                                                                                                                                      
-rw-r--r--   1 root root 2473 Oct 18  2024 sendmail                                                                                                                                                
                                                                                                                                                                                                   
/etc/cron.daily:                                                                                                                                                                                   
total 52                                                                                                                                                                                           
drwxr-xr-x   2 root root 4096 Mar  4 15:03 .                                                                                                                                                       
drwxr-xr-x 110 root root 4096 Apr 17 08:54 ..                                                                                                                                                      
-rwxr-xr-x   1 root root  376 Sep 16  2021 apport                                                                                                                                                  
-rwxr-xr-x   1 root root 1478 Apr  9  2020 apt-compat                                                                                                                                              
-rwxr-xr-x   1 root root  355 Dec 29  2017 bsdmainutils                                                                                                                                            
-rwxr-xr-x   1 root root 1187 Sep  5  2019 dpkg                                                                                                                                                    
-rwxr-xr-x   1 root root  377 Jan 21  2019 logrotate                                                                                                                                               
-rwxr-xr-x   1 root root 1123 Feb 25  2020 man-db                                                                                                                                                  
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder                                                                                                                                            
-rwxr-xr-x   1 root root 4574 Jul 18  2019 popularity-contest                                                                                                                                      
-rwxr-xr-x   1 root root 3302 Mar  7  2020 sendmail                                                                                                                                                
-rwxr-xr-x   1 root root  214 Jan 20  2023 update-notifier-common                                           
-rw-r--r--   1 root root  190 Mar 14  2023 popularity-contest                                                                                                                                      
-rw-r--r--   1 root root 2473 Oct 18  2024 sendmail                                                                                                                                                
                                                                                                                                                                                                   
/etc/cron.daily:                                                                                                                                                                                   
total 52                                                                                                                                                                                           
drwxr-xr-x   2 root root 4096 Mar  4 15:03 .                                                                                                                                                       
drwxr-xr-x 110 root root 4096 Apr 17 08:54 ..                                                                                                                                                      
-rwxr-xr-x   1 root root  376 Sep 16  2021 apport                                                                                                                                                  
-rwxr-xr-x   1 root root 1478 Apr  9  2020 apt-compat                                                                                                                                              
-rwxr-xr-x   1 root root  355 Dec 29  2017 bsdmainutils                                                                                                                                            
-rwxr-xr-x   1 root root 1187 Sep  5  2019 dpkg                                                                                                                                                    
-rwxr-xr-x   1 root root  377 Jan 21  2019 logrotate                                                                                                                                               
-rwxr-xr-x   1 root root 1123 Feb 25  2020 man-db                                                                                                                                                  
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder                                                                                                                                            
-rwxr-xr-x   1 root root 4574 Jul 18  2019 popularity-contest                                                                                                                                      
-rwxr-xr-x   1 root root 3302 Mar  7  2020 sendmail                                                                                                                                                
-rwxr-xr-x   1 root root  214 Jan 20  2023 update-notifier-common   
```

```sh
╔══════════╣ Processes with credentials in memory (root req)                                                                                                                                       
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#credentials-from-process-memory                                                                                  
gdm-password Not Found                                                                                                                                                                             
gnome-keyring-daemon Not Found                                                                                                                                                                     
lightdm Not Found                                                                                                                                                                                  
vsftpd Not Found                                                                                                                                                                                   
apache2 Not Found                                                                                                                                                                                  
sshd: process found (dump creds from memory as root)                                                                                                                                               
mysql process found (dump creds from memory as root)                                                                                                                                               
postgres Not Found                                                                                                                                                                                 
redis-server Not Found                                                                                                                                                                             
mongod Not Found                                                                                                                                                                                   
memcached Not Found                                                                                                                                                                                
elasticsearch Not Found                                                                                                                                                                            
jenkins Not Found                                                                                                                                                                                  
tomcat Not Found                                                                                                                                                                                   
nginx process found (dump creds from memory as root)                                                                                                                                               
php-fpm process found (dump creds from memory as root)                                                                                                                                             
supervisord Not Found                                                                                                                                                                              
vncserver Not Found                                                                                                                                                                                
xrdp Not Found                                                                                                                                                                                     
teamviewer Not Found   
```

use the same credintials to get in the internel admin panel then there is POC and gg.