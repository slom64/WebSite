


we will start with nmap:
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 02:c8:a4:ba:c5:ed:0b:13:ef:b7:e7:d7:ef:a2:9d:92 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJW1WZr+zu8O38glENl+84Zw9+Dw/pm4IxFauRRJ+eAFkuODRBg+5J92dT0p/BZLMz1wZMjd6BLjAkB1LHDAjqQ=
|   256 53:ea:be:c7:07:05:9d:aa:9f:44:f8:bf:32:ed:5c:9a (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICE6UoMGXZk41AvU+J2++RYnxElAD3KNSjatTdCeEa1R
80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD
|_http-title: Browsed
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

