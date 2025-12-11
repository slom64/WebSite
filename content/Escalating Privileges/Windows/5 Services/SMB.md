### List Shares
```powershell
dir \\dc01.inlanefreight.local\shares # list shares in remote server
```

### mount SMB
```sh
mount -t cifs //10.10.10.134/backups /mnt -o user=,password=    # mount SMB
```

---
### smbclient
```sh
smbclient //192.168.1.1/SharedFiles -U "Username"
smbclient //192.168.1.1/SharedFiles -U "hexdump.lab/Username%Password"

#kerberos
impacket.smbclient -k dc.rustykey.htb #--> commands --> shares, Use IT, ls, 
```

`smbclient` gives you prompt to naviage the file system:-

| Command                               | Description                       |
| ------------------------------------- | --------------------------------- |
| ls                                    | List files                        |
| cd                                    | Move between directories          |
| get                                   | Download files                    |
| put                                   | Upload file                       |
| shares                                | List all shares                   |
| Use "shareName"                       | select this share to work with it |
| recurse ON <br>prompt OFF <br> mget * |                                   |
