
Try use FQDN instead of ip in some cases, espically when dealing with kerberose. Because it like hosts names.
```sh

nxc smb 10.10.11.60 -k -u f.frizzle -p Jenni_Luvs_Magic23 XXX

nxc smb frizzdc.frizz.htb -k -u f.frizzle -p Jenni_Luvs_Magic23  <---
```


---

Put the FQDN of dc at the start of the line in  `/etc/host`.
```sh
10.10.11.60 frizzdc.frizz.htb frizzdc frizz.htb
```