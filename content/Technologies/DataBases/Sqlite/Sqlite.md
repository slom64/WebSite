```sqlite
# Open the database
sqlite3 database.db
sqlite3 database.db .dump > dump.sql

# Basic commands inside sqlite3:
.tables                    # List all tables
.schema                    # Show database schema
.schema table_name         # Show schema for specific table
.databases                 # Show attached databases
.indexes                   # List indexes
.help                      # Show all commands


-- Select all from table
SELECT * FROM users;

-- Select specific columns
SELECT username, password FROM users;

-- With conditions
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE username = 'admin';

-- Pattern matching
SELECT * FROM users WHERE username LIKE '%admin%';

-- Limit results
SELECT * FROM users LIMIT 10;
```