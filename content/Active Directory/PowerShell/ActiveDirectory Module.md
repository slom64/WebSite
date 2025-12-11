# PowerShell Active Directory Module Cheat Sheet

This cheat sheet focuses on the `ActiveDirectory` module in PowerShell, commonly used for manual enumeration in controlled environments like HackTheBox (HTB) machines. It covers key cmdlets for querying and enumerating AD objects such as users, groups, computers, domains, and more. Always ensure you have proper permissions and are operating in a legal, authorized context.

**Prerequisites:**
- Import the module: `Import-Module ActiveDirectory` (often auto-loaded on domain-joined systems).
- Authentication: Use `-Credential` parameter if needed, e.g., `-Credential (Get-Credential)`.
- Server targeting: Use `-Server <DC hostname or IP>` to specify a domain controller.
- Common filters: Use `-Filter` (e.g., `-Filter "Name -like '*admin*'"`) or `-LDAPFilter` for advanced queries.
- Output control: Pipe to `Select-Object` for specific properties, e.g., `| Select Name, SamAccountName`.
- Error handling: Add `-ErrorAction SilentlyContinue` to suppress errors.

## 1. Domain and Forest Information
Cmdlets for high-level domain/forest enumeration.

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `Get-ADDomain` | Retrieves domain details like name, NetBIOS, DNS root, FSMO roles. | `Get-ADDomain -Identity example.com \| Select Name, NetBIOSName, DomainMode` |
| `Get-ADForest` | Gets forest-wide info including domains, sites, and global catalogs. | `Get-ADForest -Identity example.com \| Select Name, RootDomain, ForestMode` |
| `Get-ADDomainController` | Lists domain controllers, including roles and site info. | `Get-ADDomainController -Filter * \| Select Hostname, IPv4Address, Site` |
| `Get-ADTrust` | Enumerates domain trusts. | `Get-ADTrust -Filter * \| Select Source, Target, Direction, TrustType` |
| `Get-ADReplicationSite` | Lists AD sites. | `Get-ADReplicationSite -Filter * \| Select Name, Location` |
| `Get-ADReplicationSubnet` | Lists subnets associated with sites. | `Get-ADReplicationSubnet -Filter * \| Select Name, Site` |

## 2. User Enumeration
Cmdlets for querying user accounts.

| Cmdlet                            | Description                                                     | Example                                                                                                                                   |
| --------------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-ADUser`                      | Retrieves user objects. Use `-Properties *` for all attributes. | `Get-ADUser -Filter * -Properties * \| Select SamAccountName, GivenName, Surname, Enabled, LastLogon`                                     |
| `Get-ADUser -SearchBase`          | Search in a specific OU.                                        | `Get-ADUser -Filter "Name -like '*service*'" -SearchBase "OU=Users,DC=example,DC=com"`                                                    |
| `Search-ADAccount`                | Finds accounts by criteria like locked, expired, inactive.      | `Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00 \| Select Name, LastLogonDate`                                                   |
| `Get-ADUser -LDAPFilter`          | Advanced LDAP queries.                                          | `Get-ADUser -LDAPFilter "(& (objectClass=user)(servicePrincipalName=*))" \| Select Name, servicePrincipalName` (for Kerberoastable users) |
| `Get-ADAccountAuthorizationGroup` | Gets groups a user is effectively member of (including nested). | `Get-ADAccountAuthorizationGroup -Identity username`                                                                                      |
```ldap
$searcher = New-Object System.DirectoryServices.DirectorySearcher
$searcher.Filter = "(&(objectCategory=person)(objectClass=user))"
$searcher.PageSize = 1000
$results = $searcher.FindAll()

foreach ($result in $results) {
    $username = $result.Properties["samaccountname"][0]
    $displayName = $result.Properties["displayname"][0]
    $email = $result.Properties["mail"][0]
    Write-Output "$username | $displayName | $email"
}
```

## 3. Group Enumeration
Cmdlets for groups and memberships.

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `Get-ADGroup` | Retrieves group objects. | `Get-ADGroup -Filter * \| Select Name, GroupCategory, GroupScope` |
| `Get-ADGroupMember` | Lists members of a group (recursive with `-Recursive`). | `Get-ADGroupMember -Identity "Domain Admins" -Recursive \| Select Name, ObjectClass` |
| `Get-ADPrincipalGroupMembership` | Gets all groups a user/principal belongs to. | `Get-ADPrincipalGroupMembership -Identity username \| Select Name` |
| `Get-ADGroup -LDAPFilter` | Advanced group queries. | `Get-ADGroup -LDAPFilter "(member:1.2.840.113556.1.4.1941:=CN=User,CN=Users,DC=example,DC=com)"` (groups containing a user, recursive) |

## 4. Computer Enumeration
Cmdlets for machine accounts.

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `Get-ADComputer` | Retrieves computer objects. | `Get-ADComputer -Filter * -Properties * \| Select Name, DNSHostName, OperatingSystem, LastLogon` |
| `Get-ADComputer -LDAPFilter` | Advanced filters, e.g., for unconstrained delegation. | `Get-ADComputer -LDAPFilter "(userAccountControl:1.2.840.113556.1.4.803:=524288)" \| Select Name` |
| `Test-ComputerSecureChannel` | Checks if the secure channel to the domain is working. | `Test-ComputerSecureChannel -Server dc.example.com` |

## 5. Organizational Units (OUs) and Objects
Cmdlets for structural enumeration.

| Cmdlet                     | Description                                     | Example                                                                                         |
| -------------------------- | ----------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `Get-ADOrganizationalUnit` | Lists OUs.                                      | `Get-ADOrganizationalUnit -Filter * \| Select Name, DistinguishedName`                          |
| `Get-ADObject`             | Generic object retrieval (users, groups, etc.). | `Get-ADObject -Filter "objectClass -eq 'organizationalUnit'" \| Select Name, DistinguishedName` |
| `Get-ADObject -SearchBase` | Search within a base DN.                        | `Get-ADObject -SearchBase "DC=example,DC=com" -Filter *`                                        |

## 6. Group Policy Objects (GPOs)
Cmdlets for GPO enumeration (requires GroupPolicy module, but often used alongside).

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `Get-GPO` | Retrieves GPOs. | `Get-GPO -All \| Select DisplayName, GpoStatus` |
| `Get-GPOReport` | Generates reports on GPOs (XML/HTML). | `Get-GPOReport -Name "Default Domain Policy" -ReportType Html -Path report.html` |
| `Get-GPInheritance` | Shows GPO inheritance for an OU/domain. | `Get-GPInheritance -Target "DC=example,DC=com"` |

## 7. Access Control and Permissions
Cmdlets for ACLs and permissions (useful for privilege escalation enumeration).

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `Get-ADObject -Properties nTSecurityDescriptor` | Gets security descriptors. | `(Get-ADObject -Identity "CN=Users,DC=example,DC=com" -Properties nTSecurityDescriptor).nTSecurityDescriptor.Access \| Select IdentityReference, ActiveDirectoryRights` |
| `Get-ACL` | Retrieves ACLs (from DirectoryServices namespace). | `(Get-Acl -Path "AD:\DC=example,DC=com").Access \| Select IdentityReference, AccessControlType` |

## 8. Password Policy and Account Info
Cmdlets for policies and sensitive account data.

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `Get-ADDefaultDomainPasswordPolicy` | Gets domain password policy. | `Get-ADDefaultDomainPasswordPolicy \| Select ComplexityEnabled, MinPasswordLength, PasswordHistoryCount` |
| `Get-ADFineGrainedPasswordPolicy` | Lists fine-grained policies. | `Get-ADFineGrainedPasswordPolicy -Filter * \| Select Name, Precedence` |
| `Get-ADUser -Properties PasswordLastSet, PasswordNeverExpires` | Checks password expiration. | `Get-ADUser -Filter * -Properties PasswordLastSet, PasswordNeverExpires \| Select Name, PasswordLastSet, PasswordNeverExpires` |

## 9. Service Principal Names (SPNs)
Useful for Kerberoasting enumeration.

| Cmdlet                                            | Description          | Example                                                                                                                         |
| ------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `Get-ADUser -Properties servicePrincipalName`     | Users with SPNs.     | `Get-ADUser -Filter {servicePrincipalName -like "*"} -Properties servicePrincipalName \| Select Name, servicePrincipalName`     |
| `Get-ADComputer -Properties servicePrincipalName` | Computers with SPNs. | `Get-ADComputer -Filter {servicePrincipalName -like "*"} -Properties servicePrincipalName \| Select Name, servicePrincipalName` |
|                                                   |                      |                                                                                                                                 |
```
# LDAP search + powershell for SPNs
$searcher = New-Object System.DirectoryServices.DirectorySearcher
$searcher.Filter = "(servicePrincipalName=*)"
$searcher.PropertiesToLoad.Add("servicePrincipalName")
$searcher.PropertiesToLoad.Add("sAMAccountName")
$searcher.PropertiesToLoad.Add("name")
$searcher.PropertiesToLoad.Add("userAccountControl")

$results = $searcher.FindAll()
foreach ($result in $results) {
    $spns = $result.Properties["servicePrincipalName"]
    $user = $result.Properties["sAMAccountName"][0]
    foreach ($spn in $spns) {
        Write-Host "$user : $spn"
    }
}
```


## 10. Advanced Queries and Tips
- **Delegation Enumeration:** For constrained/unconstrained delegation.
  - Unconstrained: `Get-ADUser -Filter {msDS-AllowedToDelegateTo -like "*"} -Properties msDS-AllowedToDelegateTo`
  - Constrained: `Get-ADComputer -Filter {userAccountControl -band 0x100000} -Properties TrustedForDelegation`
- **AS-REP Roasting:** Users with "Do not require Kerberos preauthentication."
  - `Get-ADUser -Filter {UserAccountControl -band 0x400000} -Properties UserAccountControl \| Select Name`
- **Paging Results:** For large domains, use `-ResultPageSize 1000`.
- **Exporting Data:** Pipe to `Export-Csv -Path output.csv`.
- **Combining with Other Tools:** In HTB, combine with tools like BloodHound for graphing, or PowerView for extended functionality (though this cheat sheet sticks to native AD module).

For more details, refer to official Microsoft documentation on the ActiveDirectory module. Practice in lab environments only.