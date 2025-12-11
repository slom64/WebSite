Best resource: https://www.hvs-consulting.de/en/blog/nfs-security-identifying-and-exploiting-misconfigurations

```sh
sudo showmount --all 192.168.100.2
sudo showmount -e 192.168.100.2
sudo nfs_analyze 192.168.100.2
sudo nfs_analyze 192.168.100.2  --check-no-root-squash --check-v4 --no-ping
nfs_analyze 172.18.0.1 /srv/web.fries.htb --check-no-root-squash
sudo fuse_nfs /home/slom/HTB/windows/hard/Fries/mnt  172.18.0.1  --fake-uid --fake-uid-allow-root --allow-write --manual-fh 0100070001000a00000000008a01da16c18a400cbc9b37e3567d3fba
```

> [!Danger] 
> Always use `sudo` when dealing with mounts.

---
### Linux
## Security Misconfigurations
- no_root_squash enabled
- Shares exported to (everyone)
- Writable shares
- No authentication (NFSv3)
- Sensitive directories exported
- No access restrictions by IP
- NFSv2/v3 in use (use NFSv4)
- No Kerberos authentication
- Excessive permissions on files

```sh
sudo mount -t nfs -o vers=3,nolock 192.168.100.2:/srv/web.fries.htb ./mnt
sudo mount -t nfs -o vers=4.1 192.168.100.2:/srv/web.fries.htb ./mnt
sudo mount -t nfs -o vers=3,nolock,anonuid=0,anongid=0 192.168.100.2:/srv/web.fries.htb ./mnt
```

---
### windows
- Windows supports NFS shares, but it's not a default feature and needs to be enabled
```sh
sudo mount -t nfs 10.10.11.78:/MirageReports /home/slom/HTB/windows/hard/Mirage/mnt -o nolock,vers=3 # mount nfs
sudo mount -t ntfs-3g /dev/nvme0n1p5 /mnt                       # mount ntfs
guestmount --add /mnt/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt2/ # vhd
```
