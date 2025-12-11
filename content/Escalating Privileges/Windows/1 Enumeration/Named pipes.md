```powershell
# From sysInternals
pipelist.exe /accepteula # List all named pipes.

# powersehll
Get-ChildItem \\.\pipe\

# Check which pipe we have permissions on.
accesschk.exe -w \pipe\* -v
accesschk.exe -w \pipe\* -v | findstr /v /i "spoolss lsass ntsvcs samr browser epmapper atsvc keysvc InitShutdown winreg eventlog PlugPlay rpcss winregpipe"
```
