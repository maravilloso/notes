MariaDB / MySQL
===============
# Version
```sql
  SELECT @@version;
/* 10.6.4-MariaDB */
  SELECT VERSION(); 
/* 10.6.4-MariaDB */
```
# Listar esquemas, tablas, columnas, índices y/o estadísticas
```sql
/* Listar tablas con mas de 4000 registros */
  SELECT table_name,
         table_rows
  FROM   information_schema.tables
  WHERE  table_schema = 'djezzy'
         AND table_name NOT LIKE 'django%'
         AND table_rows > 4000
  ORDER  BY 2 DESC ;
  
/* Listar FKs de una tabla */
  SELECT 
    TABLE_NAME,COLUMN_NAME,CONSTRAINT_NAME, REFERENCED_TABLE_NAME,REFERENCED_COLUMN_NAME
  FROM
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE
  WHERE
    REFERENCED_TABLE_SCHEMA = 'djezzy' AND
    TABLE_NAME = 'contracts_subscriptionpackageservice';
  
/* Detalle de columnas de una tabla */
  SELECT table_name, column_name, column_default, is_nullable, column_type, column_key, extra 
  FROM information_schema.columns
  WHERE table_schema = 'dbss' and table_name='ordered_contracts_orderedcustomer'
  ORDER BY table_name,ordinal_position
  LIMIT 50 ;
  
/* TODAS las tablas y columnas para facilitar comparar modelos y encontrar diferencias */
  SELECT table_name, column_name
  FROM information_schema.columns
  WHERE table_schema = 'dbss' 
  ORDER BY 1,2;
  
/* Buscar por fragmento de nombre de columna */
  SELECT table_name,column_name,column_default,is_nullable,column_type,column_key,extra 
  FROM information_schema.columns 
  WHERE table_schema = 'djezzy' AND column_name LIKE 'code%' 
  ORDER BY table_name, ordinal_position;
  
/* Indices de una tabla */
  SELECT DISTINCT
      TABLE_NAME,
      INDEX_NAME
  FROM INFORMATION_SCHEMA.STATISTICS
  WHERE TABLE_NAME = 'contracts_customer';
  
/* Tablas de mayor tamaño (en bytes qe ocupan en disco) ordenadas de más a menos  */
  SELECT
    TABLE_NAME, TABLE_ROWS,
    ROUND((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024) AS MB
  FROM
    information_schema.TABLES
  WHERE
    TABLE_SCHEMA = "djezzy"
  ORDER BY
    (DATA_LENGTH + INDEX_LENGTH)
  DESC;
  
/* Otra manera de calcular el tamaño exacto en bytes que ocupa una tabla */
  SELECT 
      CONCAT(FORMAT(DAT/POWER(1024,pw1),2),' ',SUBSTR(units,pw1*2+1,2)) DATSIZE,
      CONCAT(FORMAT(NDX/POWER(1024,pw2),2),' ',SUBSTR(units,pw2*2+1,2)) NDXSIZE,
      CONCAT(FORMAT(TBL/POWER(1024,pw3),2),' ',SUBSTR(units,pw3*2+1,2)) TBLSIZE
  FROM
  (
      SELECT DAT,NDX,TBL,IF(px>4,4,px) pw1,IF(py>4,4,py) pw2,IF(pz>4,4,pz) pw3
      FROM 
      (
          SELECT data_length DAT,index_length NDX,data_length+index_length TBL,
          FLOOR(LOG(IF(data_length=0,1,data_length))/LOG(1024)) px,
          FLOOR(LOG(IF(index_length=0,1,index_length))/LOG(1024)) py,
          FLOOR(LOG(data_length+index_length)/LOG(1024)) pz
          FROM information_schema.tables
          WHERE table_schema='djezzy'
          AND table_name='fourpane_memo'
      ) AA
  ) A,(SELECT 'B KBMBGBTB' units) B;
```
# Fechas y horas
```sql
/* Current date & time in UTC (Zulu) format (including milliseconds) */
  SELECT CONCAT(SUBSTRING(DATE_FORMAT(NOW(3), '%Y-%m-%dT%H:%i:%s.%f'),1,23),'Z') AS ISODT
/* 2023-12-20T09:15:34.651Z */
  
/* Exactly 6 months before current date/time */
  SELECT DATE_SUB(NOW(), INTERVAL 6 MONTH);
```
# Analizar cómo de útil es cada índice de una base de datos
  [Fuente](http://joinfu.com/presentations/kill-mysql-performance/kill-mysql-performance.odp)

```sql
  SELECT
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    s.INDEX_NAME,
    s.COLUMN_NAME,
    s.SEQ_IN_INDEX,
    ( SELECT MAX(SEQ_IN_INDEX)
      FROM INFORMATION_SCHEMA.STATISTICS s2
      WHERE s.TABLE_SCHEMA = s2.TABLE_SCHEMA
      AND s.TABLE_NAME = s2.TABLE_NAME
      AND s.INDEX_NAME = s2.INDEX_NAME
    ) AS `COLS_IN_INDEX`,
    s.CARDINALITY AS `CARD`,
    t.TABLE_ROWS AS `ROWS`,
    ROUND(((s.CARDINALITY / IFNULL(t.TABLE_ROWS, 0.01)) * 100), 2) AS `SEL %`
  FROM INFORMATION_SCHEMA.STATISTICS s
    INNER JOIN INFORMATION_SCHEMA.TABLES t
      ON s.TABLE_SCHEMA = t.TABLE_SCHEMA
      AND s.TABLE_NAME = t.TABLE_NAME
  WHERE t.TABLE_SCHEMA != 'mysql'
  AND t.TABLE_ROWS > 10
  AND s.CARDINALITY IS NOT NULL
  AND (s.CARDINALITY / IFNULL(t.TABLE_ROWS, 0.01)) < 1.00
  ORDER BY `SEL %`, TABLE_SCHEMA, TABLE_NAME;
```
# Más meta-información
```sql
/* Para dimensionamiento observar la columna Data_length */
  SHOW TABLE STATUS
  WHERE rows > 4000 
  and name not like 'a%'
  and name not like 'dj%'
  and name not like 'four%';
  
/* Mira la columna `Index_length` para ver si de verdad la PK se está generando `Collation` debería valer "utf8mb3_spanish_ci" para garantizar que se acogen caracteres especiales */
  SHOW TABLE STATUS LIKE 'subscribers';
  
/* Para ver posibles transacciones que siguen corriendo y a lo mejor han bloqueado registros */
  SHOW ENGINE INNODB STATUS;
```
# Truncar tabla aunque tenga FKs
```sql
  SET FOREIGN_KEY_CHECKS = 0; 
  truncate table number_stock_msisdn;
  SET FOREIGN_KEY_CHECKS = 1;
```
# Revisar tablas que tengan algún lock activo
```sql
  SHOW OPEN TABLES WHERE in_use > 0;
```
# Procesos
```sql
/* Ver/Matar Procesos */
  SHOW processlist;
  
  KILL 162;
  
/* Ver solamente procesos activos para ciertas BBDD en concreto */
  SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST 
  WHERE COMMAND != 'Sleep' 
  AND db LIKE 'l%';
```
# Tamaño adecuado para el **innodb_buffer_pool_size**
```sql
/* free -m | awk '/Mem/{print $2}' */
/* RAM = 31547 */
  
  SELECT sum(if(VARIABLE_NAME='innodb_buffer_pool_size',VARIABLE_VALUE,0) + if(VARIABLE_NAME='innodb_additional_mem_pool_size',VARIABLE_VALUE,0) + if(VARIABLE_NAME='innodb_additional_mem_pool_size',VARIABLE_VALUE,0) +if(VARIABLE_NAME='innodb_log_buffer_size',VARIABLE_VALUE,0) +if(VARIABLE_NAME='query_cache_size',VARIABLE_VALUE,0)) / (1024*1024) 'Global_buffers(MB)' FROM information_schema.global_variables ;
/* Global_buffers=145 */
  
  SELECT sum(if(VARIABLE_NAME='read_buffer_size ',VARIABLE_VALUE,0) + if(VARIABLE_NAME='read_rnd_buffer_size',VARIABLE_VALUE,0) + if(VARIABLE_NAME='sort_buffer_size',VARIABLE_VALUE,0) + if(VARIABLE_NAME='thread_stack',VARIABLE_VALUE,0) + if(VARIABLE_NAME='join_buffer_size',VARIABLE_VALUE,0) + if(VARIABLE_NAME='binlog_cache_size',VARIABLE_VALUE,0)) / (1024*1024) 'per_thred_buffers(MB)' FROM information_schema.global_variables;
/* per_thred_buffers = 2.94140625 */
  
  show global variables like 'max_connections';
/* max_connections=151 */
  
/* CMM= global_buffers + per_thred_buffer*max_connections = 145 + 2,94140625*151 = 589 MB */
  
  SELECT SUM(data_length+index_length)/(1024*1024) Total_InnoDB_Bytes FROM information_schema.tables WHERE engine='InnoDB';
/* Total_InnoDB_Bytes=0.1250 */
  
  SELECT variable_value/(1024*1024) innodb_buffer_pool_size FROM information_schema.global_variables WHERE Variable_name='innodb_buffer_pool_size';
/* 128 Mb */

/* Other MySQL system variables allocation (OMSVA) = CMM - configured innodb_buffer_pool_size = 589-128 = 461 Mb
  
    Allowable innodb_buffer_pool_size = (Physical Memory) – (10% of Physical Memory) – OMSVA = 31547 - 31547*0,1 - 461 = 27931,3 
  
    innodb_buffer_pool_size = (PM – PM*0.10 – OMSVA)/1.10 = 27931/1.1 = 25391 MB
  
    sudo vi /etc/my.cnf.d/server.cnf
    (add entry at [mysqld] section) innodb_buffer_pool_size = 25391M (and save)
    sudo systemctl restart mysql
*/
  
  SELECT @@innodb_buffer_pool_size;
/* 26709327872 */
  
  SELECT @@innodb_buffer_pool_chunk_size  ;
/* 134217728 */
```
# Quickly export a query to a CSV file FROM command line
```sql
  SELECT table1.primary_id       AS msisdn,
  ...
  INTO OUTFILE '/data/lyca/DUMPs/orders_report220804.csv'
  FIELDS TERMINATED BY '|'
  LINES TERMINATED BY '\n'
  ...
  FROM table1,
  ...
  WHERE
  ...
  ;
```
# Users / passwords
```sql
/* Create a new user with all the privileges and that can be used for remote connections as well */
  CREATE USER migration@localhost IDENTIFIED BY '????????';
  GRANT ALL ON *.* to migration@'%' IDENTIFIED BY '????????' WITH GRANT OPTION;
  FLUSH PRIVILEGES;
  
/* Calculate the hash of a password i.e. in order to define MARIADB_ROOT_PASSWORD_HASH for a container */
  SELECT PASSWORD('secreto');
```
# Stop running service FROM inside a container session
```bash
$ mysqladmin -u root -p shutdown
```