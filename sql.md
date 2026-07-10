| camp | purpose | why it matters |


### SQLite

#### Connecting

| Command | What It Does |
|---|---|
| `sqlite3 database.db` | Open or create a database file |
| `.tables` | List all tables |
| `.schema <table>` | Show CREATE statement for a table |
| `.mode column` | Format output as aligned columns |
| `.headers on` | Show column names in query output |
| `.quit` | Exit the SQLite shell |

#### Common Queries

| Query | What It Does |
|---|---|
| `SELECT * FROM table LIMIT 10;` | Preview first 10 rows |
| `SELECT COUNT(*) FROM table;` | Count all rows |
| `SELECT * FROM table WHERE col='val';` | Filter rows by value |
| `UPDATE table SET col='val' WHERE id=1;` | Update a specific row |
| `PRAGMA table_info(table);` | Show columns, types, and constraints |

#### Backup & Maintenance

| Command | What It Does |
|---|---|
| `sqlite3 db.db ".backup backup.db"` | Hot backup while the service is running |
| `sqlite3 db.db "PRAGMA integrity_check;"` | Check for corruption |
| `sqlite3 db.db "VACUUM;"` | Reclaim disk space, defragment the file |
| `sqlite3 db.db ".dump" > dump.sql` | Export entire database as SQL text |
| `sqlite3 new.db < dump.sql` | Restore from a SQL dump |

