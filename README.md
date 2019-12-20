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
