---
os: linux
status: Active
tags:
  - gpg
  - file_upload
  - laravel
aliases:
---
[[Web/File Upload/shells/php]]
# Resolution summary

>[!summary]
>- Step 1
>- Step 2

## Improved

- Running in mantincae mode means if you managed to do error in any page, it will revel where the error happened which is good to leak code.
- Skill 2

## Used tools

- nmap
- gobuster

# Information Gathering

Intial Nmap:

```sh
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u5 (protocol 2.0)
| ssh-hostkey: 
|   256 5c:02:33:95:ef:44:e2:80:cd:3a:96:02:23:f1:92:64 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGrihP7aP61ww7KrHUutuC/GKOyHifRmeM070LMF7b6vguneFJ3dokS/UwZxcp+H82U2LL+patf3wEpLZz1oZdQ=
|   256 1f:3d:c2:19:55:28:a1:77:59:51:48:10:c4:4b:74:ab (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ7xeTjQWBwI6WERkd6C7qIKOCnXxGGtesEDTnFtL2f2
80/tcp open  http    syn-ack ttl 63 nginx 1.22.1
| http-methods: 
|_  Supported Methods: POST
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```

Enumerated All ports:

```sh

```

Enumerated top 200 UDP ports:

```sh

```

---

# Enumeration

## Port 80 - HTTP (Apache)
```sh
gobuster dir -u http://environment.htb -w /snap/seclists/current/Discovery/Web-Content/directory-list-2.3-medium.txt


/login                (Status: 200) [Size: 2391]
/storage              (Status: 301) [Size: 169] [--> http://environment.htb/storage/] Forbiden
Progress: 597 / 220560 (0.27%)[ERROR] context deadline exceeded (Client.Timeout or context cancellation while reading body)
/up                   (Status: 200) [Size: 2125]
/logout               (Status: 302) [Size: 358] [--> http://environment.htb/login]
/vendor               (Status: 301) [Size: 169] [--> http://environment.htb/vendor/]
/build                (Status: 301) [Size: 169] [--> http://environment.htb/build/]
/mailing              (Status: 405) [Size: 244854]

```

PHP 8.2.28 — Laravel 11.30.0

![[Pasted image 20250807071824.png]]

sqlite (0.87 ms)

```sql
select * from "sessions" where "id" = 'xqQUEz8vNdA5KgUbIQUxos9ledPK2sKYX8BYlNOd' limit 1
```

![[Pasted image 20250807072827.png]]

### database

```
INSERT INTO users VALUES(1,'Hish','hish@environment.htb',NULL,'$2y$12$QPbeVM.u7VbN9KCeAJ.JA.WfWQVWQg0LopB9ILcC7akZ.q641r1gi',NULL,'2025-01-07 01:51:54','2025-01-12 01:01:48','hish.png');
INSERT INTO users VALUES(2,'Jono','jono@environment.htb',NULL,'$2y$12$i.h1rug6NfC73tTb8XF0Y.W0GDBjrY5FBfsyX2wOAXfDWOUk9dphm',NULL,'2025-01-07 01:52:35','2025-01-07 01:52:35','jono.png');
INSERT INTO users VALUES(3,'Bethany','bethany@environment.htb',NULL,'$2y$12$6kbg21YDMaGrt.iCUkP/s.yLEGAE2S78gWt.6MAODUD3JXFMS13J.',NULL,'2025-01-07 01:53:18','2025-01-07 01:53:18','bethany.png')
```

```
        'mysql' => [                                                                                                                                                                               
            'driver' => 'mysql',                                                                                                                                                                   
            'url' => env('DB_URL'),                                                                                                                                                                
            'host' => env('DB_HOST', '127.0.0.1'),                                                                                                                                                 
            'port' => env('DB_PORT', '3306'),                                                                                                                                                      
            'database' => env('DB_DATABASE', 'laravel'),                                                                                                                                           
            'username' => env('DB_USERNAME', 'root'),                                                                                                                                              
            'password' => env('DB_PASSWORD', ''),                                                                                                                                                  
            'unix_socket' => env('DB_SOCKET', ''),                                                                                                                                                 
            'charset' => env('DB_CHARSET', 'utf8mb4'),                                                                                                                                             
            'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),                                                                                                                              
            'prefix' => '',                                                                                                                                                                        
            'prefix_indexes' => true,                                                                                                                                                              
            'strict' => true,                                                                                                                                                                      
            'engine' => null,                                                                                                                                                                      
            'options' => extension_loaded('pdo_mysql') ? array_filter([                                                                                                                            
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),                                                                                                                                
            ]) : [],                                                                                                                                                                               
        ],                                                                                                                                                                                         
                                                                                                                                                                                                   
        'mariadb' => [                                                                                                                                                                             
            'driver' => 'mariadb',                                                                                                                                                                 
            'url' => env('DB_URL'),                                                                                                                                                                
            'host' => env('DB_HOST', '127.0.0.1'),                                                                                                                                                 
            'port' => env('DB_PORT', '3306'),                                                                                                                                                      
            'database' => env('DB_DATABASE', 'laravel'),                                                                                                                                           
            'username' => env('DB_USERNAME', 'root'),                                                                                                                                              
            'password' => env('DB_PASSWORD', ''),                                                                                                                                                  
            'unix_socket' => env('DB_SOCKET', ''),                                                                                                                                                 
            'charset' => env('DB_CHARSET', 'utf8mb4'),                                                                                                                                             
            'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],

        'pgsql' => [
            'driver' => 'pgsql',
            'url' => env('DB_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '5432'),
            'database' => env('DB_DATABASE', 'laravel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => env('DB_CHARSET', 'utf8'),
            'prefix' => '',
            'prefix_indexes' => true,
            'search_path' => 'public',
            'sslmode' => 'prefer',
        ],

```

```
instance.uuid  25445ac7-d4b2-4a28-b8b7-ed14aca35fd3
```


## gpg

```
www-data@environment:/home/hish$ ls -al
total 36
drwxr-xr-x 5 hish hish 4096 Apr 11 00:51 .
drwxr-xr-x 3 root root 4096 Jan 12 11:51 ..
lrwxrwxrwx 1 root root    9 Apr  7 19:29 .bash_history -> /dev/null
-rw-r--r-- 1 hish hish  220 Jan  6 21:28 .bash_logout
-rw-r--r-- 1 hish hish 3526 Jan 12 14:42 .bashrc
drwxr-xr-x 4 hish hish 4096 May  7 21:48 .gnupg
drwxr-xr-x 3 hish hish 4096 Jan  6 21:43 .local
-rw-r--r-- 1 hish hish  807 Jan  6 21:28 .profile
drwxr-xr-x 2 hish hish 4096 Jan 12 11:49 backup
-rw-r--r-- 1 root hish   33 May  7 21:46 user.txt
```

the keyvault.gpg is encrypted using the `.gnupg` folder in the home directory.

```sh
www-data@environment:/home/hish/backup$ ls -al
total 12
drwxr-xr-x 2 hish hish 4096 Jan 12 11:49 .
drwxr-xr-x 5 hish hish 4096 Apr 11 00:51 ..
-rw-r--r-- 1 hish hish  430 May  7 21:48 keyvault.gpg
```

```sh
#move it to place you have write access
cp -r /home/hish/.gnupg /tmp/mygnupg

# 2. change perm
chmod -R 700 /tmp/mygnupg

# 3. extract secret keys
gpg --homedir /tmp/mygnupg --list-secret-keys

# 4. decrypt keyvault.gpg
gpg --homedir /tmp/mygnupg --output /tmp/message.txt --decrypt /home/hish/backup/keyvault.gpg
```

Now you can read `message.txt`
```sh
www-data@environment:/tmp$ cat message.txt 
PAYPAL.COM -> Ihaves0meMon$yhere123
ENVIRONMENT.HTB -> marineSPm@ster!!
FACEBOOK.COM -> summerSunnyB3ACH!!
```

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