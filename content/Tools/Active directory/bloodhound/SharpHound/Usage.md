PowerShell version
```powershell
. .\SharpHound.ps1    # Make sure to put dot before .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -Domain Controller.local -zipFileName loot.zip
```