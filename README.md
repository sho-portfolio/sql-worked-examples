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



## JSON data (2 different formats shown below) (mysql)

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


## fill in blanks with previous non-null value (sql server)

```sql

-- https://mysqlserverteam.com/json_table-the-best-of-both-worlds/
-- https://stackoverflow.com/questions/1345065/sql-query-replace-null-value-in-a-row-with-a-value-from-the-previous-known-value
DECLARE @Table TABLE(
        ID INT,
        Val INT
)

INSERT INTO @Table (ID,Val) SELECT 1, 3
INSERT INTO @Table (ID,Val) SELECT 2, NULL
INSERT INTO @Table (ID,Val) SELECT 3, 5
INSERT INTO @Table (ID,Val) SELECT 4, NULL
INSERT INTO @Table (ID,Val) SELECT 5, NULL
INSERT INTO @Table (ID,Val) SELECT 6, 2


SELECT  *,
        ISNULL(Val, (SELECT TOP 1 Val FROM @Table WHERE ID < t.ID AND Val IS NOT NULL ORDER BY ID DESC))
FROM    @Table t
```


## turn a column of data into a comma seperated list and assign to a variable

```sql

-- https://www.aspsnippets.com/Articles/Select-Column-values-as-Comma-Separated-Delimited-string-in-SQL-Server-using-COALESCE.aspx
-- https://stackoverflow.com/questions/887628/convert-multiple-rows-into-one-with-comma-as-separator

IF OBJECT_ID(N'tempdb..#T') IS NOT NULL DROP TABLE #T
CREATE TABLE #T (col VARCHAR(10))
INSERT INTO #T(col) SELECT 'A'
INSERT INTO #T(col) SELECT 'B'
INSERT INTO #T(col) SELECT 'C'

DECLARE @S AS VARCHAR(MAX)

SELECT @S = COALESCE(@S + ',', '') + col
FROM #T

PRINT @S

```

## get size of all tables in a given database

```sql

create table #TableSize (
    Name varchar(255),
    [rows] int,
    reserved varchar(255),
    data varchar(255),
    index_size varchar(255),
    unused varchar(255))
create table #ConvertedSizes (
    Name varchar(255),
    [rows] int,
    reservedKb int,
    dataKb int,
    reservedIndexSize int,
    reservedUnused int)

EXEC sp_MSforeachtable @command1="insert into #TableSize
EXEC sp_spaceused '?'"
insert into #ConvertedSizes (Name, [rows], reservedKb, dataKb, reservedIndexSize, reservedUnused)
select name, [rows], 
SUBSTRING(reserved, 0, LEN(reserved)-2), 
SUBSTRING(data, 0, LEN(data)-2), 
SUBSTRING(index_size, 0, LEN(index_size)-2), 
SUBSTRING(unused, 0, LEN(unused)-2)
from #TableSize

select * from #ConvertedSizes
order by reservedKb desc

--drop table #TableSize
--drop table #ConvertedSizes

```


## pivot data in mysql & sql server

note that mysql produced inconsistent results when attempted ith mutual fund data
http://www.artfulsoftware.com/infotree/qrytip.php?id=78




