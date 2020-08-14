## create a table of dates (mysql)
```sql
-- not sure if the three lines below are needed
SET SESSION sql_mode = 'TRADITIONAL';
SET @@SESSION.sql_mode = 'TRADITIONAL';
SET @@sql_mode = 'TRADITIONAL';

SET SESSION cte_max_recursion_depth = 5000;

SET @startDate = '2015-01-01';	-- SAME AS: SELECT @startDate := '2019-01-01';
SET @endDate = '2019-12-31';		-- SAME AS: SELECT @endDate := '2019-01-10';

DROP TABLE IF EXISTS tblDates;
CREATE TABLE tblDates(d DATE);

INSERT INTO tblDates(d)
WITH RECURSIVE my_cte AS
(
  SELECT @startDate AS n
  UNION ALL
  SELECT DATE_ADD(n, INTERVAL 1 DAY) FROM my_cte WHERE n < @endDate
)
SELECT n FROM my_cte;

SELECT * FROM tblDates;

DROP TABLE tblDates;


-- Error Code: 3636. Recursive query aborted after 1001 iterations. 
-- Try increasing @@cte_max_recursion_depth to a larger value.


```

## create a table of dates (sql server)

```sql 

DECLARE @startDate AS DATE
DECLARE @endDate AS DATE

SET @startDate = '2015-01-01';	-- SAME AS: SELECT @startDate := '2019-01-01';
SET @endDate = '2019-12-31';		-- SAME AS: SELECT @endDate := '2019-01-10';

DROP TABLE IF EXISTS #tblDates;
CREATE TABLE #tblDates(d DATE);


WITH  my_cte AS
(
  SELECT @startDate AS n
  UNION ALL
  SELECT DATEADD(DAY, 1, n) FROM my_cte WHERE n < @endDate
)
INSERT INTO #tblDates(d)
SELECT n FROM my_cte OPTION (MAXRECURSION 10000);

SELECT * FROM #tblDates;

DROP TABLE #tblDates;
```



## query JSON data (2 different formats shown below) (mysql)

```sql

-- https://mysqlserverteam.com/json_table-the-best-of-both-worlds/

DROP TABLE IF EXISTS t1;
CREATE TABLE t1(json_col JSON);
INSERT INTO t1 VALUES (
    '[
        { "name":"John Smith",  "address":"780 Mission St, San Francisco, CA 94103"}, 
        { "name":"Sally Brown",  "address":"75 37th Ave S, St Cloud, MN 94103"}, 
        { "name":"John Johnson",  "address":"1262 Roosevelt Trail, Raymond, ME 04071"}
     ]'
);
SELECT * FROM t1;

SELECT A.* 
FROM t1, 
     JSON_TABLE(json_col, '$[*]' COLUMNS (
                name VARCHAR(40)  PATH '$.name',
                address VARCHAR(100) PATH '$.address')
		) A;


DROP TABLE IF EXISTS t1;
CREATE TABLE t1(json_col JSON);

INSERT INTO t1 VALUES (
    '{ "people": [
        { "name":"John Smith",  "address":"780 Mission St, San Francisco, CA 94103"}, 
        { "name":"Sally Brown",  "address":"75 37th Ave S, St Cloud, MN 94103"}, 
        { "name":"John Johnson",  "address":"1262 Roosevelt Trail, Raymond, ME 04071"}
     ] }'
);

SELECT A.* 
FROM t1, 
     JSON_TABLE(json_col, '$.people[*]' COLUMNS (
                name VARCHAR(40)  PATH '$.name',
                address VARCHAR(100) PATH '$.address')
     ) A;
```
