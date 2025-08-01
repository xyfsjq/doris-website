---
{
    "title": "RETENTION",
    "language": "en"
}
---

## RETENTION 

RETENTION

### Description
#### Syntax

`retention(event1, event2, ... , eventN);`

The `retention` function takes as arguments a set of conditions from 1 to 32 arguments of type `UInt8` that indicate whether a certain condition was met for the event. Any condition can be specified as an argument.

The conditions, except the first, apply in pairs: the result of the second will be true if the first and second are true, of the third if the first and third are true, etc.

To put it simply, the first digit of the return value array indicates whether `event1` is true or false, the second digit represents the truth and falseness of `event1` and `event2`, and the third digit represents whether `event1` is true or false and `event3` is true False and, and so on. If `event1` is false, return an array full of zeros.

#### Arguments

`event` — An expression that returns a `UInt8` result (1 or 0).

##### Returned value

An array of 1s and 0s with a maximum length of 32 bits, the final output array has the same length as the input parameter.

1 — Condition was met for the event.

0 — Condition wasn’t met for the event.

### example

```sql
DROP TABLE IF EXISTS retention_test;

CREATE TABLE retention_test(
                `uid` int COMMENT 'user id', 
                `date` datetime COMMENT 'date time' 
                )
DUPLICATE KEY(uid) 
DISTRIBUTED BY HASH(uid) BUCKETS 3 
PROPERTIES ( 
    "replication_num" = "1"
); 

INSERT into retention_test (uid, date) values (0, '2022-10-12'),
                                        (0, '2022-10-13'),
                                        (0, '2022-10-14'),
                                        (1, '2022-10-12'),
                                        (1, '2022-10-13'),
                                        (2, '2022-10-12'); 

SELECT * from retention_test;

+------+---------------------+
| uid  | date                |
+------+---------------------+
|    0 | 2022-10-14 00:00:00 |
|    0 | 2022-10-13 00:00:00 |
|    0 | 2022-10-12 00:00:00 |
|    1 | 2022-10-13 00:00:00 |
|    1 | 2022-10-12 00:00:00 |
|    2 | 2022-10-12 00:00:00 |
+------+---------------------+

SELECT 
    uid,     
    retention(date = '2022-10-12')
        AS r 
            FROM retention_test 
            GROUP BY uid 
            ORDER BY uid ASC;

+------+------+
| uid  | r    |
+------+------+
|    0 | [1]  | 
|    1 | [1]  |
|    2 | [1]  |
+------+------+

SELECT 
    uid,     
    retention(date = '2022-10-12', date = '2022-10-13')
        AS r 
            FROM retention_test 
            GROUP BY uid 
            ORDER BY uid ASC;

+------+--------+
| uid  | r      |
+------+--------+
|    0 | [1, 1] |
|    1 | [1, 1] |
|    2 | [1, 0] |
+------+--------+

SELECT 
    uid,     
    retention(date = '2022-10-12', date = '2022-10-13', date = '2022-10-14')
        AS r 
            FROM retention_test 
            GROUP BY uid 
            ORDER BY uid ASC;

+------+-----------+
| uid  | r         |
+------+-----------+
|    0 | [1, 1, 1] |
|    1 | [1, 1, 0] |
|    2 | [1, 0, 0] |
+------+-----------+

```

### keywords

RETENTION
