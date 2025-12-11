If you have access to server which host the CA, you can view ADCS policy.
```
reg query "HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\fries-DC01-CA\PolicyModules"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\fries-DC01-CA\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy"
```
