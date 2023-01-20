### SQL Cheatsheet

---
###### PSQL CONNECT

```C:\Program Files (x86)\pgAdmin 4\v4\runtime>  .\psql.exe 'postgresql://[USER]:[PWD]@[HOST]:[PORT]/[DB_NAME]' ```

EXAMPLE:
```C:\Program Files (x86)\pgAdmin 4\v4\runtime>  .\psql.exe 'postgresql://postgres:####@localhost:1234/db’```

---
###### POSTGRES GENERATE UUID VALID V4 
```sql
SELECT UUID_GENERATE_V4();
```
or 
```sql
SELECT uuid_in(overlay(overlay(md5(random()::text || ':' || clock_timestamp()::text) placing '4' from 13) placing to_hex(floor(random()*(11-8+1) + 8)::int)::text from 17)::cstring);
```

---
###### PG DUMP EXEMPLE:
Command line: open in terminal ```C:\Program Files (x86)\pgAdmin 4\v4\runtime```

Export only inserts from table to a file
pgsql>
```.\pg_dump.exe -p [port] -U [user] -f path\location.sql --column-inserts --data-only --table=[schema].[table] ```

export full table 
```$ ```

---
###### PG ENUM Types complete:
SQL query complete> 
```sql
SELECT
   n.nspname as schema,
   t.typname as type,
   en.*
FROM
   pg_type t
   LEFT JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace
   JOIN pg_enum en ON en.enumtypid = t.oid
WHERE
   (t.typrelid = 0
      OR (SELECT c.relkind = 'c' FROM pg_catalog.pg_class c WHERE c.oid = t.typrelid ))
   AND NOT EXISTS( SELECT 1 FROM pg_catalog.pg_type el WHERE el.oid = t.typelem AND el.typarray = t.oid)
   AND n.nspname NOT IN ('pg_catalog', 'information_schema') --and t.typname = 'customTypeName'
```
   
SQL query SHORT way>
```sql
SELECT
   t.typname AS enumtype,
   en.enumlabel AS enumlabel
FROM pg_type t
LEFT JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace
JOIN pg_enum en ON en.enumtypid = t.oid
WHERE n.nspname='[SCHEMA]' AND t.typname IN ([TYPE_NAMES])
```

----
###### PG Show FK from table
SQL query complete> 
```sql
SELECT
    tc.table_schema, 
    tc.constraint_name, 
    tc.table_name, 
    kcu.column_name, 
    ccu.table_schema AS foreign_table_schema,
    ccu.table_name AS foreign_table_name,
    ccu.column_name AS foreign_column_name 
FROM 
    information_schema.table_constraints AS tc 
    JOIN information_schema.key_column_usage AS kcu
      ON tc.constraint_name = kcu.constraint_name
      AND tc.table_schema = kcu.table_schema
    JOIN information_schema.constraint_column_usage AS ccu
      ON ccu.constraint_name = tc.constraint_name
      AND ccu.table_schema = tc.table_schema
WHERE tc.constraint_type = 'FOREIGN KEY' AND tc.table_name='my-table';
```

SQL query SHORT way>
```sql
SELECT
   conname,
   pg_catalog.pg_get_constraintdef(r.oid, true) as condef
FROM
   pg_catalog.pg_constraint r
WHERE
   r.conrelid = 'my-schema.my-table' :: regclass
   AND r.contype = 'f'
```

---
###### PG Show column name
SQL query>
```sql
SELECT
   count COLUMN_NAME
FROM
   INFORMATION_SCHEMA.COLUMNS
WHERE
   TABLE_NAME = 'my-table'
   AND TABLE_SCHEMA = 'my-schema'
```
   
----
###### PG Show column Structure
pgsql>
```\d table_name or \d+ table_name```

SQL query>
```sql
SELECT 
   table_name, 
   column_name, 
   data_type,
   character_maximum_length,
   *
FROM 
   information_schema.columns
WHERE 
   table_name = 'photo_type';
```
----

###### SQL Substring update
```sql
-- Start from 1 the text lenth
SELECT 
	substring(s.name, 1, 1) as part1,  
	substring(s.name, 2) as part2
FROM MyTable s
```