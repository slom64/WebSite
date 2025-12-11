```sh
chmod 600 ~/.ssh/mykey 
ssh-keygen -y -f ~/.ssh/mykey > ~/.ssh/mykey.pub
chmod 644 ~/.ssh/mykey.pub
```


Looking at ssh key files.
```sh
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```
You may encounter ssh keys that is encryted:
```sh
cat /home/jsmith/.ssh/SSH.private

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2109D25CC91F8DBFCEB0F7589066B2CC
```
use john to crack it.
```sh
ssh2john.py SSH.private > ssh.hash
```