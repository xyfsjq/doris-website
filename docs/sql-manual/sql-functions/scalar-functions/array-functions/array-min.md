---
{
    "title": "ARRAY_MIN",
    "language": "en"
}
---

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

## description

Get the minimum element in an array (`NULL` values are skipped).
When the array is empty or all elements in the array are `NULL` values, the function returns `NULL`.

## Syntax
```sql
ARRAY_MIN(<arr>)
```

## Parameters

| Parameter | Description |
| --- | --- |
| `<arr>` | ARRAY array |

## Return Value

Returns the smallest element in the array. Special cases:
- `NULL` values in the array are skipped.
- For an empty array or an array where all elements are `NULL`, the result is `NULL`.

## example

```sql
create table array_type_table(
    k1 INT, 
    k2 Array<int>
    ) 
duplicate key (k1)
distributed by hash(k1) buckets 1 
properties(
    'replication_num' = '1'
    );
insert into array_type_table values (0, []), (1, [NULL]), (2, [1, 2, 3]), (3, [1, NULL, 3]);
select k2, array_min(k2) from array_type_table;
```
```text
+--------------+-----------------+
| k2           | array_min(`k2`) |
+--------------+-----------------+
| []           |            NULL |
| [NULL]       |            NULL |
| [1, 2, 3]    |               1 |
| [1, NULL, 3] |               1 |
+--------------+-----------------+
```
