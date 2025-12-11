```
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 nginx
|_http-title: Did not follow redirect to http://monitorsfour.htb/
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
```

```
[21:35:28] 200 -   97B  - /.env
[21:35:29] 403 -  548B  - /.ht_wsr.txt
[21:35:29] 403 -  548B  - /.htaccess.bak1
[21:35:29] 403 -  548B  - /.htaccess.orig
[21:35:29] 403 -  548B  - /.htaccess.save
[21:35:29] 403 -  548B  - /.htaccess.sample
[21:35:29] 403 -  548B  - /.htaccess_extra
[21:35:29] 403 -  548B  - /.htaccessBAK
[21:35:29] 403 -  548B  - /.htaccess_orig
[21:35:29] 403 -  548B  - /.htaccess_sc
[21:35:29] 403 -  548B  - /.htaccessOLD2
[21:35:29] 403 -  548B  - /.htaccessOLD
[21:35:29] 403 -  548B  - /.htm
[21:35:29] 403 -  548B  - /.html
[21:35:29] 403 -  548B  - /.htpasswd_test
[21:35:29] 403 -  548B  - /.htpasswds
[21:35:29] 403 -  548B  - /.httr-oauth
[21:35:48] 200 -  367B  - /contact
[21:35:48] 403 -  548B  - /controllers/
[21:35:59] 200 -    4KB - /login
[21:36:12] 301 -  162B  - /static  ->  http://monitorsfour.htb/static/
[21:36:15] 200 -   35B  - /user
[21:36:16] 301 -  162B  - /views  ->  http://monitorsfour.htb/views/
```


```sh
DB_HOST=mariadb
DB_PORT=3306
DB_NAME=monitorsfour_db
DB_USER=monitorsdbuser
DB_PASS=f37p2j8f4t0r

admin@monitorsfour.htb : wonderful1
```

```
[{"id":2,"username":"admin","email":"admin@monitorsfour.htb","password":"56b32eb43e6f15395f6c46c1c9e1cd36","role":"super user","token":"8024b78f83f102da4f","name":"Marcus Higgins","position":"System Administrator","dob":"1978-04-26","start_date":"2021-01-12","salary":"320800.00"},

{"id":5,"username":"mwatson","email":"mwatson@monitorsfour.htb","password":"69196959c16b26ef00b77d82cf6eb169","role":"user","token":"0e543210987654321","name":"Michael Watson","position":"Website Administrator","dob":"1985-02-15","start_date":"2021-05-11","salary":"75000.00"},

{"id":6,"username":"janderson","email":"janderson@monitorsfour.htb","password":"2a22dcf99190c322d974c8df5ba3256b","role":"user","token":"0e999999999999999","name":"Jennifer Anderson","position":"Network Engineer","dob":"1990-07-16","start_date":"2021-06-20","salary":"68000.00"},

{"id":7,"username":"dthompson","email":"dthompson@monitorsfour.htb","password":"8d4a7e7fd08555133e056d9aacb1e519","role":"user","token":"0e111111111111111","name":"David Thompson","position":"Database Manager","dob":"1982-11-23","start_date":"2022-09-15","salary":"83000.00"}]
```

```
Discovered open port 3306/tcp on 172.18.0.3 
Discovered open port 3306/tcp on 172.18.0.1 
Discovered open port 80/tcp on 172.18.0.1 
Discovered open port 80/tcp on 172.18.0.2 
Discovered open port 111/tcp on 172.18.0.1 
Discovered open port 9000/tcp on 172.18.0.2
```


```
DB_NAME=monitorsfour_db
DB_USER=monitorsdbuser
DB_PASS=f37p2j8f4t0r

$database_type     = 'mysql';
$database_default  = 'cacti';
$database_username = 'cactidbuser';
$database_password = '7pyrf6ly8qx4';
```