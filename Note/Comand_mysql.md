* Sắp xếp dữ liệu theo ORDER BY

```
+------+-----------------------------------+----------------+-------------+
| id   | JOB_NAME                          | EXECUTION_TIME | INSERT_DATE |
+------+-----------------------------------+----------------+-------------+
|    1 | batch.job.close.market            |             55 | 2020-07-26  |
|    2 | batch.job.open.market             |             31 | 2020-07-26  |
|    3 | batch.job.dump.payment            |          11798 | 2020-07-26  |
|    4 | batch.job.dump.customer           |          11879 | 2020-07-26  |
|    5 | batch.job.dump.wallet             |            185 | 2020-07-26  |
+------+-----------------------------------+----------------+-------------+
```

* Sắp xếp dữ liệu tăng dần

`SELECT * FROM satoshi_report.BATCH_TIME ORDER BY JOB_NAME ASC;`

* Sắp xếp dữ liệu giảm dần

`SELECT * FROM satoshi_report.BATCH_TIME ORDER BY JOB_NAME DESC;`

`SELECT * FROM satoshi_report.BATCH_TIME ORDER BY EXECUTION_TIME DESC, JOB_NAME ASC;`

`select * from satoshi_report..BATCH_TIME WHERE EXECUTION_TIME > 10000 AND INSERT_DATE > '2021-07-26';`

* GROUP BY

```
SELECT INSERT_DATE, AVG(EXECUTION_TIME) FROM satoshi_report.BATCH_TIME GROUP BY INSERT_DATE;
SELECT INSERT_DATE, SUM(EXECUTION_TIME) FROM satoshi_report.BATCH_TIME GROUP BY INSERT_DATE;
SELECT INSERT_DATE, MIN(EXECUTION_TIME) FROM satoshi_report.BATCH_TIME GROUP BY INSERT_DATE;
SELECT INSERT_DATE, MAX(EXECUTION_TIME) FROM satoshi_report.BATCH_TIME GROUP BY INSERT_DATE;
```

```
SELECT INSERT_DATE FROM satoshi_report.BATCH_TIME GROUP BY INSERT_DATE ORDER BY INSERT_DATE DESC;
SELECT INSERT_DATE FROM satoshi_report.BATCH_TIME GROUP BY INSERT_DATE ORDER BY INSERT_DATE ASC;
```

* INSERT INTO

```
INSERT INTO BATCH_TIME(id, JOB_NAME, EXECUTION_TIME, INSERT_DATE) VALUES (%(id, %(JOB_NAME), %(EXECUTION_TIME), (INSERT_DATE));
INSERT INTO BATCH_TIME(id, JOB_NAME, EXECUTION_TIME, INSERT_DATE) VALUES (%(id, %(JOB_NAME), %(EXECUTION_TIME), (INSERT_DATE)), (%(id, %(JOB_NAME), %(EXECUTION_TIME), (INSERT_DATE)); 
```

* DELETE

`DELETE FROM `SAT_COMMON`.`SYSTEM_CONFIG` WHERE DOMAIN = 'COMMON' and CONFIG_KEY = 'system.reuter.sleep';`

* IN, BETWEEN

`SELECT * FROM satoshi_report.BATCH_TIME WHERE JOB_NAME IN('batch.job.close.market', 'batch.job.dump.wallet');`

`SELECT * FROM satoshi_report.BATCH_TIME WHERE EXECUTION_TIME BETWEEN 10000 AND 11000;`

* EXPLAIN ANALYZE

