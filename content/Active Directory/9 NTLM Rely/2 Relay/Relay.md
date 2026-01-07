This phase contain:
- sending to target server negotiation
- sending the challenge to the client.
- sending the response (Authentication message) to server.
- Using socks is easy. ntlmrealyx works as proxy, and it can know which session you want to use by putting the user you want to access like `proxychains -q smbexec.py 'INLANEFREIGHT/PETER@172.16.117.50' -no-pass -dc-ip 172.16.117.3`

We must find machines that meet some criteria: if we target `SMB` on the relay targets, we need `SMB` `signing` to be `disabled`
```sh
nxc smb 172.16.117.0/24 --gen-relay-list relay_list.txt
```

Cross-protocol

| **Relay Authentication From** | **Relay Authentication Over**                                                      | **Cross-protocol?** |
| :---------------------------- | :--------------------------------------------------------------------------------- | :------------------ |
| `HTTP`(`S`)                   | `HTTP`(`S`)                                                                        | `No`                |
| `HTTP`(`S`)                   | `IMAP` \| `LDAP`(`S`) \| `MSSQL` \| `RPC` \| `SMBv/1/2/3` \| `SMTP`                | `Yes`               |
| `SMBv/1/2/3`                  | `SMBv/1/2/3`                                                                       | `No`                |
| `SMBv/1/2/3`                  | `HTTP`(`S`) \| `IMAP` \| `LDAP`(`S`) \| `MSSQL` \| `RPC` \| `SMTP`                 | `Yes`               |
| `WCF`                         | `HTTP`(`S`) \| `IMAP` \| `LDAP`(`S`) \| `MSSQL` \| `RPC` \| `SMBv/1/2/3` \| `SMTP` | `Yes`               |

|**Source (Incoming)**|**Target (Outgoing)**|**Status**|**Why?**|
|---|---|---|---|
|**HTTP**|**LDAP/S**|✅ **BEST**|HTTP doesn't sign/bind SPN often. Best for AD Escalation.|
|**HTTP**|**MSSQL**|✅ **Valid**|Works well.|
|**SMB**|**SMB**|✅ **Valid**|Works if Signing is OFF.|
|**SMB**|**LDAP**|❌ **Fail**|**SPN Mismatch.** The SMB client hardcodes `cifs/attacker` in the blob. LDAP rejects it.|
|**MSSQL**|**Anything**|❌ **Fail**|Hard to coerce + Client hardcodes `MSSQL/attacker` SPN.|
|**LDAP**|**Anything**|❌ **Fail**|Hard to coerce + Client hardcodes `LDAP/attacker` SPN.|


> [!NOTE]
> It is important to note that if we attack a computer and receive authentication from it, we cannot relay its authentication because Windows patched the `NTLM self-relay` attack on all modern systems.
> 
> In this example, `NPORTS` also initiates authentication from `172.16.117.60`, which may cause `ntlmrelayx` to stop receiving new requests from other IPs. We can avoid this by either turning off the HTTP server using the `--no-http-server` option to prevent NPORTS's HTTP connections, editing the `Responder.conf` to not respond to requests from 172.16.117.60 or using `-tf` instead of a single target, to enable `multi-relay`.


![[Z Assets/Images/698c5ea5f8a0eb7abf93c20654f77e3a_MD5.png]]

![[Z Assets/Images/e048ff4a7860da9b76e5035d1cf8c164_MD5.png]]


---


> [!Attention] 
> Don't let responder use ports that ntlmrelayx use, because ntlmrelayx will be listening for any incoming connections in SMB, LDAP...etc, so it can do relay thing, but responder is used only to poison the network, which make other entities connect to our ntlmrelayx server, or you can use it to farm hashes.
