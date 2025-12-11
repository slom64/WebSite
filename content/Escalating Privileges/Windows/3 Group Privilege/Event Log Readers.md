This group has privileges to read event log of the system, looking at the event log could be tedious because there is alot of data. But the key is enhance your search for specific event logs like searching for succufl

```powershell
Get-WinEvent -FilterHashTable @{logname='security',id=''}  | Get-WinEventData | Select TimeCreated, e_CommandLine | ft -autosize -wrap

Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}

```
The `id` represents the event id in windows
- `4740`: A User account was locked out. "Useful in Domain joined computers"
-  `4688`: A new process has been created. "Useful in Non-Domain joined computers" , and those commands contain `/user` which may contain password.


---

```powershell
PS C:\htb> wevtutil qe Security /rd:true /f:text | Select-String "/user"

        Process Command Line:   net use T: \\fs01\backups /user:tim MyStr0ngP@ssword
        

# Passing Credentials to wevtutil
C:\htb> wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"

```


> [!NOTE]
> Searching the `Security` event log with `Get-WInEvent` requires administrator access or permissions adjusted on the registry key `HKLM\System\CurrentControlSet\Services\Eventlog\Security`. Membership in just the `Event Log Readers` group is not sufficient.