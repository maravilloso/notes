DuckDB
=====

- Import many gzipped CSV files in one shot into a new table, with custom delimiter and including a column which holds the file name the row was read from:
```sql
  CREATE TABLE pru AS SELECT * 
  FROM read_csv_auto('out/p_recharge_events2209*.csv.gz', delim='~', header=True, union_by_name=True, filename=True);
  
  select created_at, filename from pru limit 10;
  ┌─────────────────────────┬────────────────────────────────────┐
  │       created_at        │              filename              │
  │        timestamp        │              varchar               │
  ├─────────────────────────┼────────────────────────────────────┤
  │ 2022-09-23 11:52:00.788 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 18:07:47.103 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 14:46:39.82  │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 16:05:23.905 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 11:49:52.864 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 10:57:08.567 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 07:49:22.273 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 19:10:02.542 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 17:09:33.117 │ out/p_recharge_events220923.csv.gz │
  │ 2022-09-23 09:33:50.008 │ out/p_recharge_events220923.csv.gz │
  ├─────────────────────────┴────────────────────────────────────┤
  │ 10 rows                                            2 columns │
  └──────────────────────────────────────────────────────────────┘
```
- Export a query result to a CSV file:
  
```sql
  COPY (SELECT * FROM tbl) TO 'output.csv' (HEADER, DELIMITER ',');
```
- Parse gzipped JSON file/s and query its nested contents as you wish:
  
```sql
  SELECT json_extract(pd,'$[0].attributes.date-of-birth') AS birth
  FROM   (SELECT payload.data AS pd
          FROM   read_json_auto('individuals.json.gz') LIMIT 10); 
```