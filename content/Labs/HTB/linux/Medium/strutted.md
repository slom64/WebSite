---
os: linux
status: 
tags: 
aliases:
---
# Resolution summary

>[!summary]
>- Step 1
>- Step 2

## Improved skills

- Skill 1
- Skill 2

## Used tools

- nmap
- gobuster


---

# Information Gathering

Scanned initial TCP ports:

```sh
PORT   STATE SERVICE REASON         VERSION                                                                                                                                                        
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)                                                                                                  
| ssh-hostkey:                                                                                                                                                                                     
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://strutted.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```

---

# Enumeration

## Port 80 - HTTP (Apache)

You will find this is usfull articel https://y4tacker-github-io.translate.goog/2024/12/16/year/2024/12/Apache-Struts2-%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E9%80%BB%E8%BE%91%E7%BB%95%E8%BF%87-CVE-2024-53677-S2-067/?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp

and make sure to have 'Upload' not 'upload'.
and upload the file as ../../shell.jsp



```

  <user username="admin" password="IT14d6SSP81k" roles="manager-gui,admin-gui"/>  james
  Files (scripts) in /etc/profile.d/ 
  

```


privesc is easy, tcpdump as root we have doc it.

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