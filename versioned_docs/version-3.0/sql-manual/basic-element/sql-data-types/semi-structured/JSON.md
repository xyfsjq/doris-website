---
{
    "title": "JSON",
    "language": "en"
}
---

## JSON

The JSON data type stores [JSON](https://www.rfc-editor.org/rfc/rfc8785) data efficiently in a binary format and allows access to its internal fields through JSON functions.

By default, it supports up to 1048576 bytes (1MB), and can be increased up to 2147483643 bytes (2GB). This can be adjusted via the `string_type_length_soft_limit_bytes` configuration.

Compared to storing JSON strings in a regular STRING type, the JSON type has two main advantages:
1. JSON format validation during data insertion.
2. More efficient binary storage format, enabling faster access to JSON internal fields using functions like `json_extract`, compared to `get_json_xx` functions.

**Note**: In version 1.2.x, the JSON type was named JSONB. To maintain compatibility with MySQL, it was renamed to JSON starting from version 2.0.0. Older tables can still use the previous name.

### Syntax

**Definition:**
```sql
json_column_name JSON
```

**Insertion:**
- Using `INSERT INTO VALUES` with the format as a string surrounded by quotes. For example:
```sql
INSERT INTO table_name(id, json_column_name) VALUES (1, '{"k1": "100"}')
```

- For STREAM LOAD, the format for the corresponding column is a string without additional quotes. For example:
```
12	{"k1":"v31", "k2": 300}
13	[]
14	[123, 456]
```

- When the  all special character with `'\'` such as `'\r'`, `'\t'` appears in JSON, you need to use the replace function to replace `"\"` with `"\\"`, for example, you need replace `"\n"` to `"\\n"` 


**Query:**
- Directly select the entire JSON column:
```sql
SELECT json_column_name FROM table_name;
```

- Extract specific fields or other information from JSON using JSON functions. For example:
```sql
SELECT json_extract(json_column_name, '$.k1') FROM table_name;
```

- The JSON type can be cast to and from integers, strings, BOOLEAN, ARRAY, and MAP. For example:
```sql
SELECT CAST('{"k1": "100"}' AS JSON);
SELECT CAST(json_column_name AS STRING) FROM table_name;
SELECT CAST(json_extract(json_column_name, '$.k1') AS INT) FROM table_name;
```

:::tip
The JSON type currently cannot be used for `GROUP BY`, `ORDER BY`, or comparison operations.
:::

### example
A tutorial for JSON datatype including create table, load data and query.

#### create database and table

```
CREATE DATABASE testdb;

USE testdb;

CREATE TABLE test_json (
  id INT,
  j JSON
)
DUPLICATE KEY(id)
DISTRIBUTED BY HASH(id) BUCKETS 10
PROPERTIES("replication_num" = "1");
```

#### Load data

##### stream load test_json.csv test data

- there are 2 columns, the 1st column is id and the 2nd column is json string
- there are 25 rows, the first 18 rows are valid json and the last 7 rows are invalid


```
1	\N
2	null
3	true
4	false
5	100
6	10000
7	1000000000
8	1152921504606846976
9	6.18
10	"abcd"
11	{}
12	{"k1":"v31", "k2": 300}
13	[]
14	[123, 456]
15	["abc", "def"]
16	[null, true, false, 100, 6.18, "abc"]
17	[{"k1":"v41", "k2": 400}, 1, "a", 3.14]
18	{"k1":"v31", "k2": 300, "a1": [{"k1":"v41", "k2": 400}, 1, "a", 3.14]}
19	''
20	'abc'
21	abc
22	100x
23	6.a8
24	{x
25	[123, abc]
```

- due to the 28% of rows is invalid, stream load with default configuration will fail with error message "too many filtered rows"

```
curl --location-trusted -u root: -T test_json.csv http://127.0.0.1:8840/api/testdb/test_json/_stream_load
{
    "TxnId": 12019,
    "Label": "744d9821-9c9f-43dc-bf3b-7ab048f14e32",
    "TwoPhaseCommit": "false",
    "Status": "Fail",
    "Message": "too many filtered rows",
    "NumberTotalRows": 25,
    "NumberLoadedRows": 18,
    "NumberFilteredRows": 7,
    "NumberUnselectedRows": 0,
    "LoadBytes": 380,
    "LoadTimeMs": 48,
    "BeginTxnTimeMs": 0,
    "StreamLoadPutTimeMs": 1,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 45,
    "CommitAndPublishTimeMs": 0,
    "ErrorURL": "http://172.21.0.5:8840/api/_load_error_log?file=__shard_2/error_log_insert_stmt_95435c4bf5f156df-426735082a9296af_95435c4bf5f156df_426735082a9296af"
}
```

- stream load will success after set header configuration 'max_filter_ratio: 0.3'
```
curl --location-trusted -u root: -H 'max_filter_ratio: 0.3' -T test_json.csv http://127.0.0.1:8840/api/testdb/test_json/_stream_load
{
    "TxnId": 12017,
    "Label": "f37a50c1-43e9-4f4e-a159-a3db6abe2579",
    "TwoPhaseCommit": "false",
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 25,
    "NumberLoadedRows": 18,
    "NumberFilteredRows": 7,
    "NumberUnselectedRows": 0,
    "LoadBytes": 380,
    "LoadTimeMs": 68,
    "BeginTxnTimeMs": 0,
    "StreamLoadPutTimeMs": 2,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 45,
    "CommitAndPublishTimeMs": 19,
    "ErrorURL": "http://172.21.0.5:8840/api/_load_error_log?file=__shard_0/error_log_insert_stmt_a1463f98a7b15caf-c79399b920f5bfa3_a1463f98a7b15caf_c79399b920f5bfa3"
}
```

- use SELECT to view the data loaded by stream load. The column with JSON type will be displayed as plain JSON string.

```
mysql> SELECT * FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+
| id   | j                                                             |
+------+---------------------------------------------------------------+
|    1 |                                                          NULL |
|    2 |                                                          null |
|    3 |                                                          true |
|    4 |                                                         false |
|    5 |                                                           100 |
|    6 |                                                         10000 |
|    7 |                                                    1000000000 |
|    8 |                                           1152921504606846976 |
|    9 |                                                          6.18 |
|   10 |                                                        "abcd" |
|   11 |                                                            {} |
|   12 |                                         {"k1":"v31","k2":300} |
|   13 |                                                            [] |
|   14 |                                                     [123,456] |
|   15 |                                                 ["abc","def"] |
|   16 |                              [null,true,false,100,6.18,"abc"] |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |
+------+---------------------------------------------------------------+
18 rows in set (0.03 sec)

```

##### write data using insert into

- total rows increae from 18 to 19 after insert 1 row
```
mysql> INSERT INTO test_json VALUES(26, '{"k1":"v1", "k2": 200}');
Query OK, 1 row affected (0.09 sec)
{'label':'insert_4ece6769d1b42fd_ac9f25b3b8f3dc02', 'status':'VISIBLE', 'txnId':'12016'}

mysql> SELECT * FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+
| id   | j                                                             |
+------+---------------------------------------------------------------+
|    1 |                                                          NULL |
|    2 |                                                          null |
|    3 |                                                          true |
|    4 |                                                         false |
|    5 |                                                           100 |
|    6 |                                                         10000 |
|    7 |                                                    1000000000 |
|    8 |                                           1152921504606846976 |
|    9 |                                                          6.18 |
|   10 |                                                        "abcd" |
|   11 |                                                            {} |
|   12 |                                         {"k1":"v31","k2":300} |
|   13 |                                                            [] |
|   14 |                                                     [123,456] |
|   15 |                                                 ["abc","def"] |
|   16 |                              [null,true,false,100,6.18,"abc"] |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |
|   26 |                                          {"k1":"v1","k2":200} |
+------+---------------------------------------------------------------+
19 rows in set (0.03 sec)

```

#### Query

##### extract some filed from json by json_extract functions

1. extract the whole json, '$' stands for root in json path
```
+------+---------------------------------------------------------------+---------------------------------------------------------------+
| id   | j                                                             | json_extract(`j`, '$')                                       |
+------+---------------------------------------------------------------+---------------------------------------------------------------+
|    1 |                                                          NULL |                                                          NULL |
|    2 |                                                          null |                                                          null |
|    3 |                                                          true |                                                          true |
|    4 |                                                         false |                                                         false |
|    5 |                                                           100 |                                                           100 |
|    6 |                                                         10000 |                                                         10000 |
|    7 |                                                    1000000000 |                                                    1000000000 |
|    8 |                                           1152921504606846976 |                                           1152921504606846976 |
|    9 |                                                          6.18 |                                                          6.18 |
|   10 |                                                        "abcd" |                                                        "abcd" |
|   11 |                                                            {} |                                                            {} |
|   12 |                                         {"k1":"v31","k2":300} |                                         {"k1":"v31","k2":300} |
|   13 |                                                            [] |                                                            [] |
|   14 |                                                     [123,456] |                                                     [123,456] |
|   15 |                                                 ["abc","def"] |                                                 ["abc","def"] |
|   16 |                              [null,true,false,100,6.18,"abc"] |                              [null,true,false,100,6.18,"abc"] |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                            [{"k1":"v41","k2":400},1,"a",3.14] |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |
|   26 |                                          {"k1":"v1","k2":200} |                                          {"k1":"v1","k2":200} |
+------+---------------------------------------------------------------+---------------------------------------------------------------+
19 rows in set (0.03 sec)
```

1. extract k1 field, return NULL if it does not exist
```
mysql> SELECT id, j, json_extract(j, '$.k1') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+----------------------------+
| id   | j                                                             | json_extract(`j`, '$.k1') |
+------+---------------------------------------------------------------+----------------------------+
|    1 |                                                          NULL |                       NULL |
|    2 |                                                          null |                       NULL |
|    3 |                                                          true |                       NULL |
|    4 |                                                         false |                       NULL |
|    5 |                                                           100 |                       NULL |
|    6 |                                                         10000 |                       NULL |
|    7 |                                                    1000000000 |                       NULL |
|    8 |                                           1152921504606846976 |                       NULL |
|    9 |                                                          6.18 |                       NULL |
|   10 |                                                        "abcd" |                       NULL |
|   11 |                                                            {} |                       NULL |
|   12 |                                         {"k1":"v31","k2":300} |                      "v31" |
|   13 |                                                            [] |                       NULL |
|   14 |                                                     [123,456] |                       NULL |
|   15 |                                                 ["abc","def"] |                       NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                       NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                       NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                      "v31" |
|   26 |                                          {"k1":"v1","k2":200} |                       "v1" |
+------+---------------------------------------------------------------+----------------------------+
19 rows in set (0.03 sec)
```

1. extract element 0 of the top level array
```
mysql> SELECT id, j, json_extract(j, '$[0]') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+----------------------------+
| id   | j                                                             | json_extract(`j`, '$[0]') |
+------+---------------------------------------------------------------+----------------------------+
|    1 |                                                          NULL |                       NULL |
|    2 |                                                          null |                       NULL |
|    3 |                                                          true |                       NULL |
|    4 |                                                         false |                       NULL |
|    5 |                                                           100 |                       NULL |
|    6 |                                                         10000 |                       NULL |
|    7 |                                                    1000000000 |                       NULL |
|    8 |                                           1152921504606846976 |                       NULL |
|    9 |                                                          6.18 |                       NULL |
|   10 |                                                        "abcd" |                       NULL |
|   11 |                                                            {} |                       NULL |
|   12 |                                         {"k1":"v31","k2":300} |                       NULL |
|   13 |                                                            [] |                       NULL |
|   14 |                                                     [123,456] |                        123 |
|   15 |                                                 ["abc","def"] |                      "abc" |
|   16 |                              [null,true,false,100,6.18,"abc"] |                       null |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |      {"k1":"v41","k2":400} |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                       NULL |
|   26 |                                          {"k1":"v1","k2":200} |                       NULL |
+------+---------------------------------------------------------------+----------------------------+
19 rows in set (0.03 sec)
```

1. extract a whole json array of name a1
```
mysql> SELECT id, j, json_extract(j, '$.a1') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+------------------------------------+
| id   | j                                                             | json_extract(`j`, '$.a1')         |
+------+---------------------------------------------------------------+------------------------------------+
|    1 |                                                          NULL |                               NULL |
|    2 |                                                          null |                               NULL |
|    3 |                                                          true |                               NULL |
|    4 |                                                         false |                               NULL |
|    5 |                                                           100 |                               NULL |
|    6 |                                                         10000 |                               NULL |
|    7 |                                                    1000000000 |                               NULL |
|    8 |                                           1152921504606846976 |                               NULL |
|    9 |                                                          6.18 |                               NULL |
|   10 |                                                        "abcd" |                               NULL |
|   11 |                                                            {} |                               NULL |
|   12 |                                         {"k1":"v31","k2":300} |                               NULL |
|   13 |                                                            [] |                               NULL |
|   14 |                                                     [123,456] |                               NULL |
|   15 |                                                 ["abc","def"] |                               NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                               NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                               NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} | [{"k1":"v41","k2":400},1,"a",3.14] |
|   26 |                                          {"k1":"v1","k2":200} |                               NULL |
+------+---------------------------------------------------------------+------------------------------------+
19 rows in set (0.02 sec)
```

1. extract nested field from an object in an array
```
mysql> SELECT id, j, json_extract(j, '$.a1[0]'), json_extract(j, '$.a1[0].k1') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+-------------------------------+----------------------------------+
| id   | j                                                             | json_extract(`j`, '$.a1[0]') | json_extract(`j`, '$.a1[0].k1') |
+------+---------------------------------------------------------------+-------------------------------+----------------------------------+
|    1 |                                                          NULL |                          NULL |                             NULL |
|    2 |                                                          null |                          NULL |                             NULL |
|    3 |                                                          true |                          NULL |                             NULL |
|    4 |                                                         false |                          NULL |                             NULL |
|    5 |                                                           100 |                          NULL |                             NULL |
|    6 |                                                         10000 |                          NULL |                             NULL |
|    7 |                                                    1000000000 |                          NULL |                             NULL |
|    8 |                                           1152921504606846976 |                          NULL |                             NULL |
|    9 |                                                          6.18 |                          NULL |                             NULL |
|   10 |                                                        "abcd" |                          NULL |                             NULL |
|   11 |                                                            {} |                          NULL |                             NULL |
|   12 |                                         {"k1":"v31","k2":300} |                          NULL |                             NULL |
|   13 |                                                            [] |                          NULL |                             NULL |
|   14 |                                                     [123,456] |                          NULL |                             NULL |
|   15 |                                                 ["abc","def"] |                          NULL |                             NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                          NULL |                             NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                          NULL |                             NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |         {"k1":"v41","k2":400} |                            "v41" |
|   26 |                                          {"k1":"v1","k2":200} |                          NULL |                             NULL |
+------+---------------------------------------------------------------+-------------------------------+----------------------------------+
19 rows in set (0.02 sec)

```

1. extract field with specific datatype
- json_extract_string will extract field with string type, convert to string if the field is not string
```
mysql> SELECT id, j, json_extract_string(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+---------------------------------------------------------------+
| id   | j                                                             | json_extract_string(`j`, '$')                                |
+------+---------------------------------------------------------------+---------------------------------------------------------------+
|    1 | NULL                                                          | NULL                                                          |
|    2 | null                                                          | null                                                          |
|    3 | true                                                          | true                                                          |
|    4 | false                                                         | false                                                         |
|    5 | 100                                                           | 100                                                           |
|    6 | 10000                                                         | 10000                                                         |
|    7 | 1000000000                                                    | 1000000000                                                    |
|    8 | 1152921504606846976                                           | 1152921504606846976                                           |
|    9 | 6.18                                                          | 6.18                                                          |
|   10 | "abcd"                                                        | abcd                                                          |
|   11 | {}                                                            | {}                                                            |
|   12 | {"k1":"v31","k2":300}                                         | {"k1":"v31","k2":300}                                         |
|   13 | []                                                            | []                                                            |
|   14 | [123,456]                                                     | [123,456]                                                     |
|   15 | ["abc","def"]                                                 | ["abc","def"]                                                 |
|   16 | [null,true,false,100,6.18,"abc"]                              | [null,true,false,100,6.18,"abc"]                              |
|   17 | [{"k1":"v41","k2":400},1,"a",3.14]                            | [{"k1":"v41","k2":400},1,"a",3.14]                            |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |
|   26 | {"k1":"v1","k2":200}                                          | {"k1":"v1","k2":200}                                          |
+------+---------------------------------------------------------------+---------------------------------------------------------------+
19 rows in set (0.02 sec)

mysql> SELECT id, j, json_extract_string(j, '$.k1') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+-----------------------------------+
| id   | j                                                             | json_extract_string(`j`, '$.k1') |
+------+---------------------------------------------------------------+-----------------------------------+
|    1 |                                                          NULL | NULL                              |
|    2 |                                                          null | NULL                              |
|    3 |                                                          true | NULL                              |
|    4 |                                                         false | NULL                              |
|    5 |                                                           100 | NULL                              |
|    6 |                                                         10000 | NULL                              |
|    7 |                                                    1000000000 | NULL                              |
|    8 |                                           1152921504606846976 | NULL                              |
|    9 |                                                          6.18 | NULL                              |
|   10 |                                                        "abcd" | NULL                              |
|   11 |                                                            {} | NULL                              |
|   12 |                                         {"k1":"v31","k2":300} | v31                               |
|   13 |                                                            [] | NULL                              |
|   14 |                                                     [123,456] | NULL                              |
|   15 |                                                 ["abc","def"] | NULL                              |
|   16 |                              [null,true,false,100,6.18,"abc"] | NULL                              |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] | NULL                              |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} | v31                               |
|   26 |                                          {"k1":"v1","k2":200} | v1                                |
+------+---------------------------------------------------------------+-----------------------------------+
19 rows in set (0.03 sec)

```

- json_extract_int will extract field with int type, return NULL if the field is not int
```
mysql> SELECT id, j, json_extract_int(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+-----------------------------+
| id   | j                                                             | json_extract_int(`j`, '$') |
+------+---------------------------------------------------------------+-----------------------------+
|    1 |                                                          NULL |                        NULL |
|    2 |                                                          null |                        NULL |
|    3 |                                                          true |                        NULL |
|    4 |                                                         false |                        NULL |
|    5 |                                                           100 |                         100 |
|    6 |                                                         10000 |                       10000 |
|    7 |                                                    1000000000 |                  1000000000 |
|    8 |                                           1152921504606846976 |                        NULL |
|    9 |                                                          6.18 |                        NULL |
|   10 |                                                        "abcd" |                        NULL |
|   11 |                                                            {} |                        NULL |
|   12 |                                         {"k1":"v31","k2":300} |                        NULL |
|   13 |                                                            [] |                        NULL |
|   14 |                                                     [123,456] |                        NULL |
|   15 |                                                 ["abc","def"] |                        NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                        NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                        NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                        NULL |
|   26 |                                          {"k1":"v1","k2":200} |                        NULL |
+------+---------------------------------------------------------------+-----------------------------+
19 rows in set (0.02 sec)

mysql> SELECT id, j, json_extract_int(j, '$.k2') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+--------------------------------+
| id   | j                                                             | json_extract_int(`j`, '$.k2') |
+------+---------------------------------------------------------------+--------------------------------+
|    1 |                                                          NULL |                           NULL |
|    2 |                                                          null |                           NULL |
|    3 |                                                          true |                           NULL |
|    4 |                                                         false |                           NULL |
|    5 |                                                           100 |                           NULL |
|    6 |                                                         10000 |                           NULL |
|    7 |                                                    1000000000 |                           NULL |
|    8 |                                           1152921504606846976 |                           NULL |
|    9 |                                                          6.18 |                           NULL |
|   10 |                                                        "abcd" |                           NULL |
|   11 |                                                            {} |                           NULL |
|   12 |                                         {"k1":"v31","k2":300} |                            300 |
|   13 |                                                            [] |                           NULL |
|   14 |                                                     [123,456] |                           NULL |
|   15 |                                                 ["abc","def"] |                           NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                           NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                           NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                            300 |
|   26 |                                          {"k1":"v1","k2":200} |                            200 |
+------+---------------------------------------------------------------+--------------------------------+
19 rows in set (0.03 sec)
```

- json_extract_bigint will extract field with bigint type, return NULL if the field is not bigint
```
mysql> SELECT id, j, json_extract_bigint(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+--------------------------------+
| id   | j                                                             | json_extract_bigint(`j`, '$') |
+------+---------------------------------------------------------------+--------------------------------+
|    1 |                                                          NULL |                           NULL |
|    2 |                                                          null |                           NULL |
|    3 |                                                          true |                           NULL |
|    4 |                                                         false |                           NULL |
|    5 |                                                           100 |                            100 |
|    6 |                                                         10000 |                          10000 |
|    7 |                                                    1000000000 |                     1000000000 |
|    8 |                                           1152921504606846976 |            1152921504606846976 |
|    9 |                                                          6.18 |                           NULL |
|   10 |                                                        "abcd" |                           NULL |
|   11 |                                                            {} |                           NULL |
|   12 |                                         {"k1":"v31","k2":300} |                           NULL |
|   13 |                                                            [] |                           NULL |
|   14 |                                                     [123,456] |                           NULL |
|   15 |                                                 ["abc","def"] |                           NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                           NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                           NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                           NULL |
|   26 |                                          {"k1":"v1","k2":200} |                           NULL |
+------+---------------------------------------------------------------+--------------------------------+
19 rows in set (0.03 sec)

mysql> SELECT id, j, json_extract_bigint(j, '$.k2') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+-----------------------------------+
| id   | j                                                             | json_extract_bigint(`j`, '$.k2') |
+------+---------------------------------------------------------------+-----------------------------------+
|    1 |                                                          NULL |                              NULL |
|    2 |                                                          null |                              NULL |
|    3 |                                                          true |                              NULL |
|    4 |                                                         false |                              NULL |
|    5 |                                                           100 |                              NULL |
|    6 |                                                         10000 |                              NULL |
|    7 |                                                    1000000000 |                              NULL |
|    8 |                                           1152921504606846976 |                              NULL |
|    9 |                                                          6.18 |                              NULL |
|   10 |                                                        "abcd" |                              NULL |
|   11 |                                                            {} |                              NULL |
|   12 |                                         {"k1":"v31","k2":300} |                               300 |
|   13 |                                                            [] |                              NULL |
|   14 |                                                     [123,456] |                              NULL |
|   15 |                                                 ["abc","def"] |                              NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                              NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                              NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                               300 |
|   26 |                                          {"k1":"v1","k2":200} |                               200 |
+------+---------------------------------------------------------------+-----------------------------------+
19 rows in set (0.02 sec)

```

- json_extract_double will extract field with double type, return NULL if the field is not double
```
mysql> SELECT id, j, json_extract_double(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+--------------------------------+
| id   | j                                                             | json_extract_double(`j`, '$') |
+------+---------------------------------------------------------------+--------------------------------+
|    1 |                                                          NULL |                           NULL |
|    2 |                                                          null |                           NULL |
|    3 |                                                          true |                           NULL |
|    4 |                                                         false |                           NULL |
|    5 |                                                           100 |                            100 |
|    6 |                                                         10000 |                          10000 |
|    7 |                                                    1000000000 |                     1000000000 |
|    8 |                                           1152921504606846976 |          1.152921504606847e+18 |
|    9 |                                                          6.18 |                           6.18 |
|   10 |                                                        "abcd" |                           NULL |
|   11 |                                                            {} |                           NULL |
|   12 |                                         {"k1":"v31","k2":300} |                           NULL |
|   13 |                                                            [] |                           NULL |
|   14 |                                                     [123,456] |                           NULL |
|   15 |                                                 ["abc","def"] |                           NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                           NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                           NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                           NULL |
|   26 |                                          {"k1":"v1","k2":200} |                           NULL |
+------+---------------------------------------------------------------+--------------------------------+
19 rows in set (0.02 sec)

mysql> SELECT id, j, json_extract_double(j, '$.k2') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+-----------------------------------+
| id   | j                                                             | json_extract_double(`j`, '$.k2') |
+------+---------------------------------------------------------------+-----------------------------------+
|    1 |                                                          NULL |                              NULL |
|    2 |                                                          null |                              NULL |
|    3 |                                                          true |                              NULL |
|    4 |                                                         false |                              NULL |
|    5 |                                                           100 |                              NULL |
|    6 |                                                         10000 |                              NULL |
|    7 |                                                    1000000000 |                              NULL |
|    8 |                                           1152921504606846976 |                              NULL |
|    9 |                                                          6.18 |                              NULL |
|   10 |                                                        "abcd" |                              NULL |
|   11 |                                                            {} |                              NULL |
|   12 |                                         {"k1":"v31","k2":300} |                               300 |
|   13 |                                                            [] |                              NULL |
|   14 |                                                     [123,456] |                              NULL |
|   15 |                                                 ["abc","def"] |                              NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                              NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                              NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                               300 |
|   26 |                                          {"k1":"v1","k2":200} |                               200 |
+------+---------------------------------------------------------------+-----------------------------------+
19 rows in set (0.03 sec)
```

- json_extract_bool will extract field with boolean type, return NULL if the field is not boolean
```
mysql> SELECT id, j, json_extract_bool(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+------------------------------+
| id   | j                                                             | json_extract_bool(`j`, '$') |
+------+---------------------------------------------------------------+------------------------------+
|    1 |                                                          NULL |                         NULL |
|    2 |                                                          null |                         NULL |
|    3 |                                                          true |                            1 |
|    4 |                                                         false |                            0 |
|    5 |                                                           100 |                         NULL |
|    6 |                                                         10000 |                         NULL |
|    7 |                                                    1000000000 |                         NULL |
|    8 |                                           1152921504606846976 |                         NULL |
|    9 |                                                          6.18 |                         NULL |
|   10 |                                                        "abcd" |                         NULL |
|   11 |                                                            {} |                         NULL |
|   12 |                                         {"k1":"v31","k2":300} |                         NULL |
|   13 |                                                            [] |                         NULL |
|   14 |                                                     [123,456] |                         NULL |
|   15 |                                                 ["abc","def"] |                         NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                         NULL |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                         NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                         NULL |
|   26 |                                          {"k1":"v1","k2":200} |                         NULL |
+------+---------------------------------------------------------------+------------------------------+
19 rows in set (0.01 sec)

mysql> SELECT id, j, json_extract_bool(j, '$[1]') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+---------------------------------+
| id   | j                                                             | json_extract_bool(`j`, '$[1]') |
+------+---------------------------------------------------------------+---------------------------------+
|    1 |                                                          NULL |                            NULL |
|    2 |                                                          null |                            NULL |
|    3 |                                                          true |                            NULL |
|    4 |                                                         false |                            NULL |
|    5 |                                                           100 |                            NULL |
|    6 |                                                         10000 |                            NULL |
|    7 |                                                    1000000000 |                            NULL |
|    8 |                                           1152921504606846976 |                            NULL |
|    9 |                                                          6.18 |                            NULL |
|   10 |                                                        "abcd" |                            NULL |
|   11 |                                                            {} |                            NULL |
|   12 |                                         {"k1":"v31","k2":300} |                            NULL |
|   13 |                                                            [] |                            NULL |
|   14 |                                                     [123,456] |                            NULL |
|   15 |                                                 ["abc","def"] |                            NULL |
|   16 |                              [null,true,false,100,6.18,"abc"] |                               1 |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                            NULL |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                            NULL |
|   26 |                                          {"k1":"v1","k2":200} |                            NULL |
+------+---------------------------------------------------------------+---------------------------------+
19 rows in set (0.01 sec)
```

- json_extract_isnull will extract field with json null type, return 1 if the field is json null , else 0
- json null is different from SQL NULL. SQL NULL stands for no value for a field, but json null stands for an field with special value null.
```
mysql> SELECT id, j, json_extract_isnull(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+--------------------------------+
| id   | j                                                             | json_extract_isnull(`j`, '$') |
+------+---------------------------------------------------------------+--------------------------------+
|    1 |                                                          NULL |                           NULL |
|    2 |                                                          null |                              1 |
|    3 |                                                          true |                              0 |
|    4 |                                                         false |                              0 |
|    5 |                                                           100 |                              0 |
|    6 |                                                         10000 |                              0 |
|    7 |                                                    1000000000 |                              0 |
|    8 |                                           1152921504606846976 |                              0 |
|    9 |                                                          6.18 |                              0 |
|   10 |                                                        "abcd" |                              0 |
|   11 |                                                            {} |                              0 |
|   12 |                                         {"k1":"v31","k2":300} |                              0 |
|   13 |                                                            [] |                              0 |
|   14 |                                                     [123,456] |                              0 |
|   15 |                                                 ["abc","def"] |                              0 |
|   16 |                              [null,true,false,100,6.18,"abc"] |                              0 |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                              0 |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                              0 |
|   26 |                                          {"k1":"v1","k2":200} |                              0 |
+------+---------------------------------------------------------------+--------------------------------+
19 rows in set (0.03 sec)

```

##### check if a field is existed in json by json_exists_path

```
mysql> SELECT id, j, json_exists_path(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+-----------------------------+
| id   | j                                                             | json_exists_path(`j`, '$') |
+------+---------------------------------------------------------------+-----------------------------+
|    1 |                                                          NULL |                        NULL |
|    2 |                                                          null |                           1 |
|    3 |                                                          true |                           1 |
|    4 |                                                         false |                           1 |
|    5 |                                                           100 |                           1 |
|    6 |                                                         10000 |                           1 |
|    7 |                                                    1000000000 |                           1 |
|    8 |                                           1152921504606846976 |                           1 |
|    9 |                                                          6.18 |                           1 |
|   10 |                                                        "abcd" |                           1 |
|   11 |                                                            {} |                           1 |
|   12 |                                         {"k1":"v31","k2":300} |                           1 |
|   13 |                                                            [] |                           1 |
|   14 |                                                     [123,456] |                           1 |
|   15 |                                                 ["abc","def"] |                           1 |
|   16 |                              [null,true,false,100,6.18,"abc"] |                           1 |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                           1 |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                           1 |
|   26 |                                          {"k1":"v1","k2":200} |                           1 |
+------+---------------------------------------------------------------+-----------------------------+
19 rows in set (0.02 sec)

mysql> SELECT id, j, json_exists_path(j, '$.k1') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+--------------------------------+
| id   | j                                                             | json_exists_path(`j`, '$.k1') |
+------+---------------------------------------------------------------+--------------------------------+
|    1 |                                                          NULL |                           NULL |
|    2 |                                                          null |                              0 |
|    3 |                                                          true |                              0 |
|    4 |                                                         false |                              0 |
|    5 |                                                           100 |                              0 |
|    6 |                                                         10000 |                              0 |
|    7 |                                                    1000000000 |                              0 |
|    8 |                                           1152921504606846976 |                              0 |
|    9 |                                                          6.18 |                              0 |
|   10 |                                                        "abcd" |                              0 |
|   11 |                                                            {} |                              0 |
|   12 |                                         {"k1":"v31","k2":300} |                              1 |
|   13 |                                                            [] |                              0 |
|   14 |                                                     [123,456] |                              0 |
|   15 |                                                 ["abc","def"] |                              0 |
|   16 |                              [null,true,false,100,6.18,"abc"] |                              0 |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                              0 |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                              1 |
|   26 |                                          {"k1":"v1","k2":200} |                              1 |
+------+---------------------------------------------------------------+--------------------------------+
19 rows in set (0.03 sec)

mysql> SELECT id, j, json_exists_path(j, '$[2]') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+--------------------------------+
| id   | j                                                             | json_exists_path(`j`, '$[2]') |
+------+---------------------------------------------------------------+--------------------------------+
|    1 |                                                          NULL |                           NULL |
|    2 |                                                          null |                              0 |
|    3 |                                                          true |                              0 |
|    4 |                                                         false |                              0 |
|    5 |                                                           100 |                              0 |
|    6 |                                                         10000 |                              0 |
|    7 |                                                    1000000000 |                              0 |
|    8 |                                           1152921504606846976 |                              0 |
|    9 |                                                          6.18 |                              0 |
|   10 |                                                        "abcd" |                              0 |
|   11 |                                                            {} |                              0 |
|   12 |                                         {"k1":"v31","k2":300} |                              0 |
|   13 |                                                            [] |                              0 |
|   14 |                                                     [123,456] |                              0 |
|   15 |                                                 ["abc","def"] |                              0 |
|   16 |                              [null,true,false,100,6.18,"abc"] |                              1 |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] |                              1 |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} |                              0 |
|   26 |                                          {"k1":"v1","k2":200} |                              0 |
+------+---------------------------------------------------------------+--------------------------------+
19 rows in set (0.02 sec)


```

##### get the datatype of a field in json by json_type

- return the data type of the field specified by json path, NULL if not existed.
```
mysql> SELECT id, j, json_type(j, '$') FROM test_json ORDER BY id;
+------+---------------------------------------------------------------+----------------------+
| id   | j                                                             | json_type(`j`, '$') |
+------+---------------------------------------------------------------+----------------------+
|    1 |                                                          NULL | NULL                 |
|    2 |                                                          null | null                 |
|    3 |                                                          true | bool                 |
|    4 |                                                         false | bool                 |
|    5 |                                                           100 | int                  |
|    6 |                                                         10000 | int                  |
|    7 |                                                    1000000000 | int                  |
|    8 |                                           1152921504606846976 | bigint               |
|    9 |                                                          6.18 | double               |
|   10 |                                                        "abcd" | string               |
|   11 |                                                            {} | object               |
|   12 |                                         {"k1":"v31","k2":300} | object               |
|   13 |                                                            [] | array                |
|   14 |                                                     [123,456] | array                |
|   15 |                                                 ["abc","def"] | array                |
|   16 |                              [null,true,false,100,6.18,"abc"] | array                |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] | array                |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} | object               |
|   26 |                                          {"k1":"v1","k2":200} | object               |
+------+---------------------------------------------------------------+----------------------+
19 rows in set (0.02 sec)

mysql> select id, j, json_type(j, '$.k1') from test_json order by id;
+------+---------------------------------------------------------------+-------------------------+
| id   | j                                                             | json_type(`j`, '$.k1') |
+------+---------------------------------------------------------------+-------------------------+
|    1 |                                                          NULL | NULL                    |
|    2 |                                                          null | NULL                    |
|    3 |                                                          true | NULL                    |
|    4 |                                                         false | NULL                    |
|    5 |                                                           100 | NULL                    |
|    6 |                                                         10000 | NULL                    |
|    7 |                                                    1000000000 | NULL                    |
|    8 |                                           1152921504606846976 | NULL                    |
|    9 |                                                          6.18 | NULL                    |
|   10 |                                                        "abcd" | NULL                    |
|   11 |                                                            {} | NULL                    |
|   12 |                                         {"k1":"v31","k2":300} | string                  |
|   13 |                                                            [] | NULL                    |
|   14 |                                                     [123,456] | NULL                    |
|   15 |                                                 ["abc","def"] | NULL                    |
|   16 |                              [null,true,false,100,6.18,"abc"] | NULL                    |
|   17 |                            [{"k1":"v41","k2":400},1,"a",3.14] | NULL                    |
|   18 | {"k1":"v31","k2":300,"a1":[{"k1":"v41","k2":400},1,"a",3.14]} | string                  |
|   26 |                                          {"k1":"v1","k2":200} | string                  |
+------+---------------------------------------------------------------+-------------------------+
19 rows in set (0.03 sec)

```

### FAQ

1. Is there a difference between null in JSON and NULL in SQL (i.e., IS NULL)?

Yes, there is a difference. In JSON, null (e.g., {"key1": null}) means that the key key1 exists and its value is explicitly null. This is a special type that gets encoded into JSON binary.

In contrast, SQL NULL (when using IS NULL) can indicate that the key doesn’t exist at all in the JSON object.

For example:

``` sql
mysql> SELECT JSON_EXTRACT_STRING('{"key1" : null}', "$.key1") IS NULL;
+----------------------------------------------------------+
| JSON_EXTRACT_STRING('{"key1" : null}', "$.key1") IS NULL |
+----------------------------------------------------------+
|                                                        0 |
+----------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT JSON_EXTRACT_STRING('{"key1" : null}', "$.key_not_exist") IS NULL;
+-------------------------------------------------------------------+
| JSON_EXTRACT_STRING('{"key1" : null}', "$.key_not_exist") IS NULL |
+-------------------------------------------------------------------+
|                                                                 1 |
+-------------------------------------------------------------------+
1 row in set (0.01 sec)
```

2. What’s the difference between `GET_JSON_XXX` and `JSON_EXTRACT_XXX` functions, and how should I choose between them?

The `GET_JSON_XXX` functions are designed for use on string types — they extract values directly from raw JSON strings. On the other hand, `JSON_EXTRACT_XXX` functions are implemented specifically for the JSON data type and are optimized for it.

### keywords
JSONB, JSON, json_parse, json_parse_error_to_null, json_parse_error_to_value, json_extract, json_extract_isnull, json_extract_bool, json_extract_int, json_extract_bigint, json_extract_double, json_extract_string, json_exists_path, json_type

