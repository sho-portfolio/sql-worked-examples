## create a table of dates (mysql)

```sql
SET @startDate = '2019-01-01';	-- SAME AS: SELECT @startDate := '2019-01-01';
SET @endDate = '2019-01-10';		-- SELECT @endDate := '2019-01-10';

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
```
