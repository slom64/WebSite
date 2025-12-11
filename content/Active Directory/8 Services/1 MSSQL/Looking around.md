```sql
Use table;
-- List all tables
SELECT * FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_CATALOG = 'financial_planner_DB';

-- Or use this alternative
SELECT name FROM financial_planner.sys.tables;

-- List all objects (tables, views, procedures)
SELECT * FROM financial_planner.sys.objects;
```