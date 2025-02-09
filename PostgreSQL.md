PostgreSQL
==========

## Version

```sql
SELECT version();
-- PostgreSQL 11.2 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-36), 64-bit
```
## Configuration / Status

```sql
SHOW config_file
-- /var/lib/pgsql/11/data/postgresql.conf
SELECT pg_reload_conf()

-- Show current settings
SHOW ALL ;
```

PostgreSQL Service status:
```bash
$ systemctl status postgresql-13
```
## Change user pass (logged as root user)

```sql
ALTER USER postgres PASSWORD 'mmlycamig';
```
## Default/current schema

```sql
SELECT current_schema();
```
## All tables in all catalogs

```sql
SELECT * FROM pg_catalog.pg_tables;
```
## DESC table

```sql
SELECT column_name, ordinal_position, is_nullable, data_type 
FROM information_schema.columns WHERE TABLE_NAME = 'individuals'
```
## All indexes in the schema

```sql
SELECT * FROM pg_indexes WHERE schemaname = 'public'
ORDER BY tablename, indexname;
```
## Sizes

```sql
-- Estimated Nº of rows of all tables in the schema
SELECT 
  nspname AS schemaname,relname,reltuples
FROM pg_class C
LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
WHERE 
  nspname NOT IN ('pg_catalog', 'information_schema') AND
  relkind='r' 
ORDER BY relname

-- Actual nº of rows
WITH tbl AS (
       SELECT table_schema,
              table_name
       FROM   information_schema.TABLES
       WHERE  table_name NOT LIKE 'pg_%'
       AND    table_schema IN ('public'))
SELECT   table_schema,
         table_name,
         (Xpath('/row/c/text()', 
          Query_to_xml(Format('select count(*) as c from %I.%I', 
          				table_schema, table_name), FALSE, TRUE, ''))
         )[1]::text::INT AS rows_n
FROM     tbl
ORDER BY 2;

-- Tables total data size
SELECT relname AS "relation",
       pg_size_pretty( pg_total_relation_size (C .oid) ) AS "total_size"
FROM pg_class C LEFT JOIN pg_namespace N ON (N.oid = C .relnamespace)
WHERE nspname NOT IN ('pg_catalog','information_schema')
AND C .relkind <> 'i' AND nspname !~ '^pg_toast'
ORDER BY pg_total_relation_size (C .oid) desc;
```
## Processes / running queries

```sql
select pid, application_name, backend_start, query_start, state_change, state, query
from pg_stat_activity
order by state_change desc nulls last
```
## pga4dash

```sql
SELECT 'session_stats' AS chart_name, row_to_json(t) AS chart_data
FROM (SELECT
   (SELECT count(*) FROM pg_stat_activity WHERE datname = (SELECT datname FROM pg_database WHERE oid = 16385)) AS "Total",
   (SELECT count(*) FROM pg_stat_activity WHERE state = 'active' AND datname = (SELECT datname FROM pg_database WHERE oid = 16385))  AS "Active",
   (SELECT count(*) FROM pg_stat_activity WHERE state = 'idle' AND datname = (SELECT datname FROM pg_database WHERE oid = 16385))  AS "Idle"
) t
UNION ALL
SELECT 'tps_stats' AS chart_name, row_to_json(t) AS chart_data
FROM (SELECT
   (SELECT sum(xact_commit) + sum(xact_rollback) FROM pg_stat_database WHERE datname = (SELECT datname FROM pg_database WHERE oid = 16385)) AS "Transactions",
   (SELECT sum(xact_commit) FROM pg_stat_database WHERE datname = (SELECT datname FROM pg_database WHERE oid = 16385)) AS "Commits",
   (SELECT sum(xact_rollback) FROM pg_stat_database WHERE datname = (SELECT datname FROM pg_database WHERE oid = 16385)) AS "Rollbacks"
) t
```