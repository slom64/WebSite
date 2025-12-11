
### list outbound connections of an account:
```powershell
# Requires RSAT ActiveDirectory module (built-in on DCs, or install on workstations)
Import-Module ActiveDirectory

# Change this to your target account
$TargetAccount = "john.doe"   # or "WEB01$" for computer

# Resolve the objectSid and DN of the target account
$Searcher = New-Object System.DirectoryServices.DirectorySearcher
$Searcher.Filter = "(|(sAMAccountName=$TargetAccount)(userPrincipalName=$TargetAccount))"
$TargetObject = $Searcher.FindOne()

if (-not $TargetObject) { Write-Error "Account not found"; break }

$TargetDN = $TargetObject.Properties.distinguishedname[0]
$TargetSID = New-Object System.Security.Principal.SecurityIdentifier($TargetObject.Properties.objectsid[0], 0)

Write-Host "Searching outbound rights for: $TargetDN" -ForegroundColor Cyan
Write-Host "SID: $($TargetSID.Value)`n" -ForegroundColor Yellow

# Security Descriptor Flags: only look at DACL (not SACL or Owner)
$SDFlags = [System.DirectoryServices.SecurityDescriptorFlag]::Dacl

# Extended rights GUIDs we care about
$ExtendedRights = @{
    "User-Force-Change-Password" = "00299570-246d-11d0-a768-00aa006e0529"
    "Send-As"                    = "ab721a53-1e2f-11d0-9819-00aa0040529b"
    "Receive-As"                 = "ab721a54-1e2f-11d0-9819-00aa0040529b"
    "Add/Remove-Self-Member"     = "bf9679c0-0de6-11d0-a285-00aa003049e2"
}

# Query the entire domain partitioning set
$DomainDN = (Get-ADDomain).DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher
$Searcher.SearchRoot = "LDAP://$DomainDN"
$Searcher.PageSize = 1000
$Searcher.PropertiesToLoad.Add("distinguishedName") | Out-Null
$Searcher.PropertiesToLoad.Add("nTSecurityDescriptor") | Out-Null
$Searcher.PropertiesToLoad.Add("objectClass") | Out-Null
$Searcher.PropertiesToLoad.Add("sAMAccountName") | Out-Null
$Searcher.PropertiesToLoad.Add("userAccountControl") | Out-Null

# Speed hack: only objects that actually have an ACL worth checking
$AllObjects = $Searcher.FindAll()

$Results = foreach ($Obj in $AllObjects) {
    $DN = $Obj.Properties.distinguishedname[0]
    $SD = New-Object System.DirectoryServices.ActiveDirectorySecurity
    try {
        $RawSD = $Obj.Properties["ntsecuritydescriptor"][0]
        $SD.SetSecurityDescriptorBinaryForm($RawSD, $SDFlags)
    } catch { continue }

    foreach ($ACE in $SD.DiscretionaryAcl) {
        if ($ACE.SecurityIdentifier.Value -eq $TargetSID.Value) {
            $Rights = $ACE.AccessControlType
            $AceType = $ACE.AceType

            # Translate dangerous rights
            $IsDangerous = $false
            $Description = ""

            switch ($ACE.ObjectAceType) {
                $ExtendedRights["User-Force-Change-Password"] { $Description = "Force Password Change"; $IsDangerous = $true }
                $ExtendedRights["Send-As"]                    { $Description = "Send-As"; $IsDangerous = $true }
                $ExtendedRights["Receive-As"]                 { $Description = "Receive-As"; $IsDangerous = $true }
                $ExtendedRights["Add/Remove-Self-Member"]     { $Description = "Add/Remove Self as Member"; $IsDangerous = $true }
            }

            if ($ACE.AccessMask -band 0x000F0000) { $IsDangerous = $true } # Extended rights bit
            if ($ACE.AccessMask -band 0x00000100) { $Description += " WriteDacl"; $IsDangerous = $true }
            if ($ACE.AccessMask -band 0x00000020) { $Description += " WriteOwner"; $IsDangerous = $true }
            if ($ACE.AccessMask -band 0x00020000) { $Description += " GenericWrite/AllProperties"; $IsDangerous = $true }
            if ($ACE.AccessMask -band 0x00010000) { $Description += " GenericAll/FullControl"; $IsDangerous = $true }

            if ($IsDangerous -or ($Description -match "WriteDacl|WriteOwner|GenericAll|Force Password|Send-As|Receive-As|Self-Member")) {
                [PSCustomObject]@{
                    TargetDN           = $DN
                    TargetType         = ($Obj.Properties.objectclass[-1])
                    sAMAccountName     = $Obj.Properties.samaccountname[0]
                    AccessRight        = $ACE.ActiveDirectoryRights
                    AccessMask         = "0x{0:X8}" -f $ACE.AccessMask
                    AceType            = $ACE.AceType
                    Inherited          = $ACE.IsInherited
                    Description        = $Description.Trim()
                }
            }
        }
    }
}

# Output results sorted by danger
$Results | Sort-Object Description, TargetDN | Format-Table -Wrap -AutoSize
```

---

### Change password of an account
```powershell


net user user2 "Newpassword123!" /domain #tested
_______________________ or
Import-Module ActiveDirectory

$User   = "user2"                    # target samAccountName or DN or UPN
$NewPass = "Newpassword123!"

# Convert to SecureString
$SecurePassword = ConvertTo-SecureString $NewPass -AsPlainText -Force

# Reset password (this is the direct equivalent of Set-DomainUserPassword)
Set-ADAccountPassword -Identity $User -NewPassword $SecurePassword -Reset -Verbose

# Optional: force user to change password at next logon
# Set-ADUser -Identity $User -ChangePasswordAtLogon $true
____________ or 
# Method 2 – Using .NET (System.DirectoryServices.AccountManagement) – No RSAT needed on modern Windows
Add-Type -AssemblyName System.DirectoryServices.AccountManagement

$User       = "user2"
$NewPass    = "Newpassword123!"
$SecurePass = ConvertTo-SecureString $NewPass -AsPlainText -Force

$Context = [System.DirectoryServices.AccountManagement.PrincipalContext]::new('Domain')
$UserPrincipal = [System.DirectoryServices.AccountManagement.UserPrincipal]::FindByIdentity($Context, 'samAccountName', $User)

$UserPrincipal.SetPassword($NewPass)   # this does the reset
# $UserPrincipal.ExpirePasswordNow()   # uncomment to force change at next logon

$UserPrincipal.Save()
$Context.Dispose()
____________or 
# method 3: ldap
$UserDN = "CN=user2,OU=Users,DC=corp,DC=local"   # <-- change to real DN
$NewPass = "Newpassword123!"

# Encode password as Unicode + quotes (LDAP requires this)
$UnicodePass = [System.Text.Encoding]::Unicode.GetBytes('"' + $NewPass + '"')
$SecurePass = New-Object System.Security.SecureString
$UnicodePass[2..($UnicodePass.Length-3)] | ForEach-Object { $SecurePass.AppendChar([char]$_) }
$SecurePass.MakeReadOnly()

$User = [ADSI]"LDAP://$UserDN"
$User.psbase.Invoke("SetPassword", $NewPass)   # simple version (requires rights)

# OR the more correct way (replaceAttribute with unicodePwd):
$User.Put("unicodePwd", $UnicodePass)
$User.SetInfo()
___________ or 

```


---
### Change UPN of account
```powershell
dsmod user "CN=user2,OU=Users,DC=lab,DC=local" -upn "user3@lab.local" # tested
__________ or

Import-Module ActiveDirectory

Set-ADUser -Identity user2 -UserPrincipalName "user3@lab.local" -Verbose
Set-ADUser -Identity "CN=user2,OU=Users,DC=lab,DC=local" -UserPrincipalName "user3@lab.local" -Verbose
___________________________ or 
# No RSAT? Use .NET (System.DirectoryServices.AccountManagement)
Add-Type -AssemblyName System.DirectoryServices.AccountManagement

$Context = [System.DirectoryServices.AccountManagement.PrincipalContext]::new('Domain') $User = [System.DirectoryServices.AccountManagement.UserPrincipal]::FindByIdentity($Context, 'user2')

$User.UserPrincipalName = "user3@lab.local" $User.Save()

$Context.Dispose()
________________________or
# ldap
([ADSI]"LDAP://CN=user2,OU=Users,DC=lab,DC=local").Put("userPrincipalName", "user3@lab.local").SetInfo()
_______________________or

```