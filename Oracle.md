Oracle
======

### Get Oracle version:

```sql
SELECT * FROM V$VERSION
```

### SQL*Plus nice layout

```sql
-- SQLPlus nice layout
set wrap off
set linesize 3000
SET PAGESIZE 200

```
### Execute script file with SQL*plus:
```sql
time nohup sqlplus -S "user/pass" @./SQL/script.sql myarg1 myarg2 &
```
note that then passed arguments can be references from inside the SQL script using `&1`, `&2`, etc.
```sql
update mytable set mycol = '&2' where myid = '&1';
```
### Enable visualizing execution plan for queries so you can check if/how indexes are used (Only works in SQL*Plus)

```sql
set autotrace trace explain;
```
### Discover the service name to be used in **tnsnames.ora** from SQL*Plus:

```sql
select value from v$parameter where name='service_names';
```
### Dumping a query result directly to a text file from SQL*Plus ([more](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqpug/generating-HTML-reports-from-SQL-Plus.html#GUID-45C7A587-A4C0-4345-96BD-2A0892AAEBEC) - [info](https://stackoverflow.com/questions/643137/how-do-i-spool-to-a-csv-formatted-file-using-sqlplus))

```sql
SET MARKUP CSV ON DELIMITER | QUOTE OFF
set termout off
set feedback off
set sqlprompt ''
set trimspool on

SPOOL myfile.csv
SELECT ...
SPOOL off;
```
### The one row Oracle dummy table DUAL:

```sql
SELECT TO_CHAR(SYSDATE,'DD/MM/RR HH24:MI:SS') FROM DUAL;
```
### Get DDL for a specific table:

```sql
SELECT DBMS_METADATA.get_ddl ('TABLE', 'BILLING_ACCOUNTS') FROM dual;
```
### Searching for tables/columns based on part of its names:

```sql
SELECT SUBSTR(table_name, 1, 30)  AS tabla, SUBSTR(column_name, 1, 30) AS campo
FROM   all_tab_columns
WHERE  owner IN ( 'REPLICA', 'QVMIG002' )
       AND table_name LIKE 'CRM_C%'
       AND column_name LIKE '%CONSU%'
```
### Table partitions:

```sql
select * from ALL_TAB_PARTITIONS where table_name = upper('nombretabla');
```
### Show table indexes:

```sql
select column_name from all_ind_columns where table_name = 'HPD_RESOUR';
```
### Create index in a concrete tablespace:

```sql
CREATE INDEX addr_fk1 ON address(cust_id) TABLESPACE TBS_INDEX;
```
### Refresh indexes/_statistics_ for table:

```sql
ANALYZE TABLE t COMPUTE STATISTICS FOR ALL INDEXES;
```
### Drop a table without sending it to the recycle bin so its space is really freed:

```sql
DROP TABLE tabla PURGE;
```
### Rename table:

```sql
ALTER TABLE TMP_HPD RENAME TO HPD_DATA ;
```
### Rename column:

```sql
ALTER TABLE RESOURCES RENAME COLUMN luw_id TO agreement_id ;
```
### See procedure source code:

```sql
SELECT text FROM all_source WHERE name = 'CREATE_BILLTARGET' ORDER BY TYPE, LINE;
```
### Find and kill open sessions (with embedded ``alter system kill session`` ... )

```sql
SELECT 'alter system kill session '||CHR(39)||s.sid||', '||s.serial#||CHR(39)||';',
--       p.spid,
       s.username,s.schemaname,s.program,
--       s.terminal,
       s.osuser
FROM   v$session s JOIN v$process p ON s.paddr = p.addr
WHERE  s.TYPE != 'BACKGROUND'
       AND osuser = 'marcos'; 
and s.program like '%Thin%';
```
### Get expensive ops from those sessions

```sql
select TO_CHAR(last_update_time, 'DD-MON-YY HH24.MI.SS') as updated, elapsed_seconds, 
        time_remaining, target, sql_plan_options, opname, sofar, totalwork
from V$SESSION_LONGOPS v
where sid in (select sid from v$session where type != 'BACKGROUND')
order by last_update_time  desc;
```
### See existing links to other databases, remote properties, list tables from those, with row count included, etc:

```sql
SELECT * FROM ALL_DB_LINKS;

select property_name, property_value 
from database_properties@bidb ;

select owner, table_name, num_rows
from all_tables@BIDB
where owner IN ('REPLICA', 'CBS')
order by owner, table_name;
```
### Merge used for fast partially update table field/s with another's:

```sql
MERGE INTO TMP_CBS c
USING (SELECT TP_CUST_KEY_NEW, msisdn FROM replica.X_BAN_CCAA_DWH) B
ON (B.msisdn=c.msisdn)
WHEN MATCHED THEN UPDATE SET c.TP_CUST_KEY=B.TP_CUST_KEY_NEW
```
### Gigabytes per schema:

```sql
select owner, round(sum(bytes)/1024/1024/1024) as GBs
from dba_segments
group by owner
order by 2 desc
```
### Gigabytes per segment type:

```sql
select segment_type, round(sum(bytes)/1024/1024/1024) as GBs
from dba_segments
group by segment_type
order by 2 desc
```
### Tablespaces sizes and available space:

```sql
select df.tablespace_name "Tablespace",
       totalusedspace "Used MB",
       (df.totalspace - tu.totalusedspace) "Free MB",
       df.totalspace "Total MB",
       round(100 * ( (df.totalspace - tu.totalusedspace)/ df.totalspace)) "Pct. Free"
  from (select tablespace_name,
               round(sum(bytes) / 1048576) TotalSpace
          from dba_data_files 
         group by tablespace_name) df,
       (select round(sum(bytes)/(1024*1024)) totalusedspace,
               tablespace_name
          from dba_segments 
         group by tablespace_name) tu
 where df.tablespace_name = tu.tablespace_name 
   and df.totalspace <> 0;
```
### Largest tables (in storage space)

```sql
SELECT owner,
       table_name,
       tablespace_name,
       num_rows,
       ROUND(blocks * 8 / 1024) AS size_mb,
       pct_free,
       compression,
       logging
FROM   all_tables
WHERE  blocks IS NOT NULL
-- and owner = USER
ORDER  BY 5 DESC ; 
```
### Backup tables to a dump file:

```shell
exp userid=qvmig002/qvantel@genesisdb tables=BKP_CIM_DEV_REG_SOT,BKP_RIM_DEV_SOT file=SoTs.dmp
```
### Load Java class and dependency JAR:

```sql
call dbms_java.grant_permission( 'RFCMIG', 'SYS:java.io.FilePermission', '/home/migration/pdi-ce-7.1.0.0-12/data-integration/libext/java-uuid-generator-3.1.0.jar', 'read' );
call dbms_java.loadjava('/home/migration/pdi-ce-7.1.0.0-12/data-integration/libext/java-uuid-generator-3.1.0.jar');

create or replace and compile
java source named "generateUUID"
as
import java.security.MessageDigest;
import com.fasterxml.uuid.impl.NameBasedGenerator;

public class generateUUID {
    private static com.fasterxml.uuid.impl.NameBasedGenerator gen;
    static {
        try {
            gen = new NameBasedGenerator(
                NameBasedGenerator.NAMESPACE_DNS, MessageDigest.getInstance("SHA-1"),
                com.fasterxml.uuid.UUIDType.NAME_BASED_SHA1);
        } catch (Exception e) {}
    }
    public static String create(String id) throws Exception {
        return gen.generate(id).toString();
    }
}
/

CREATE OR REPLACE FUNCTION generateUUID(TXT VARCHAR2)
RETURN VARCHAR2
AS LANGUAGE JAVA
NAME 'generateUUID.create(java.lang.String) return java.lang.String';
/

select GENERATEUUID('hola') from dual
-- f2f96c1c-64fa-5520-a058-85e2e0d5e0c1
```
### Maximizing table data loading speeds:
Problem: you’re loading a large amount of data into a table and want to insert new records as quickly as possible.

Solution: first, set the table’s logging attribute to ``NOLOGGING`` this minimizes the generation redo for direct path operations (this feature has no effect on regular DML operations). Then use a direct path loading feature, such as the following:

```sql
INSERT /*+ APPEND */ -- on queries that use a subquery for determining which records are inserted
INSERT /*+ APPEND_VALUES */ -- on queries that use a VALUES clause
CREATE TABLE...AS SELECT
```
To enable NOLOGGING use the ``ALTER TABLE`` statement as follows:

```sql
SQL> alter table emp NOLOGGING;
```
Or directly when creating the table:

```sql
CREATE TABLE new_customer
TABLESPACE new_ts
NOLOGGING 
AS SELECT * FROM customer;
```
### Displaying Automated Segment Advisor Advice:

```sql
SELECT 'Segment Advice --------------------------'
       || CHR(10)
       || 'TABLESPACE_NAME : '
       || tablespace_name
       || CHR(10)
       || 'SEGMENT_OWNER : '
       || segment_owner
       || CHR(10)
       || 'SEGMENT_NAME : '
       || segment_name
       || CHR(10)
       || 'ALLOCATED_SPACE : '
       || allocated_space
       || CHR(10)
       || 'RECLAIMABLE_SPACE: '
       || reclaimable_space
       || CHR(10)
       || 'RECOMMENDATIONS : '
       || recommendations
       || CHR(10)
       || 'SOLUTION 1 : '
       || c1
       || CHR(10)
       || 'SOLUTION 2 : '
       || c2
       || CHR(10)
       || 'SOLUTION 3 : '
       || c3 Advice
FROM   TABLE(dbms_space.ASA_RECOMMENDATIONS('FALSE', 'FALSE', 'FALSE'));  
```