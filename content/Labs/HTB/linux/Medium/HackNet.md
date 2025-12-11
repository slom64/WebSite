
inital nmap 
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 95:62:ef:97:31:82:ff:a1:c6:08:01:8c:6a:0f:dc:1c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ8BFa2rPKTgVLDq1GN85n/cGWndJ63dTBCsAS6v3n8j85AwatuF1UE+C95eEdeMPbZ1t26HrjltEg2Dj+1A2DM=
|   256 5f:bd:93:10:20:70:e6:09:f1:ba:6a:43:58:86:42:66 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFOSA3zBloIJP6JRvvREkPtPv013BYN+NNzn3kcJj0cH
80/tcp open  http    syn-ack ttl 63 nginx 1.22.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to http://hacknet.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

dirsearch
```
[14:45:08] 302 -    0B  - /comment  ->  /
[14:45:11] 302 -    0B  - /contacts  ->  /
[14:45:17] 302 -    0B  - /explore  ->  /
[14:45:18] 404 -  555B  - /favicon.ico
[14:45:28] 200 -  857B  - /login
[14:45:28] 302 -    0B  - /logout  ->  /
[14:45:29] 301 -  169B  - /media  ->  http://hacknet.htb/media/
[14:45:29] 404 -  555B  - /media.tar.gz
[14:45:29] 404 -  555B  - /media.tar
[14:45:29] 404 -  555B  - /media.tar.bz2
[14:45:29] 404 -  555B  - /media.zip
[14:45:29] 404 -  555B  - /media_admin
[14:45:29] 200 -  383B  - /media/
[14:45:29] 404 -  555B  - /media/export-criteo.xml
[14:45:30] 302 -    0B  - /messages  ->  /
[14:45:37] 302 -    0B  - /post  ->  /
[14:45:38] 302 -    0B  - /profile  ->  /
[14:45:39] 200 -  948B  - /register
[14:45:40] 302 -    0B  - /search  ->  /
[14:45:44] 404 -  555B  - /static/dump.sql
[14:45:44] 404 -  555B  - /static/api/swagger.json
[14:45:44] 404 -  555B  - /static/api/swagger.yaml
contacts                [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 117ms]
explore                 [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 152ms]

```


> [!NOTE]
> Seems no subdomains.
