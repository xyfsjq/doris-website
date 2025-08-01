---
{
    "title": "ARRAY_DIFFERENCE",
    "language": "zh-CN"
}
---

## 描述
计算相邻数组元素之间的差异。返回一个数组，其中第一个元素将为 0，第二个元素是 a[1]-a[0]之间的差值。
注意若 NULL 值存在，返回结果为 NULL

## 语法
```sql
ARRAY_DIFFERENCE(<arr>)
```

## 参数
| 参数 | 说明 |
|---|---|
| `<arr>` | 用于相邻数组元素之间的差异的数组 |

## 返回值
返回一个数组。特殊情况：
- 如果 NULL 值存在，返回结果为 NULL

## 举例

```sql
CREATE TABLE array_type_table (
   k1 INT,
   k2 ARRAY<INT>
)
duplicate key (k1)
distributed by hash(k1) buckets 1
properties(
  'replication_num' = '1'
);
INSERT INTO array_type_table (k1, k2) VALUES
(0, []),
(1, [NULL]),
(2, [1, 2, 3]),
(3, [1, NULL, 3]),
(4, [0, 1, 2, 3, NULL, 4, 6]),
(5, [1, 2, 3, 4, 5, 4, 3, 2, 1]),
(6, [6, 7, 8]);
select *,array_difference(k2) from array_type_table;
```
```text
+------+-----------------------------+---------------------------------+
| k1   | k2                          | array_difference(`k2`)          |
+------+-----------------------------+---------------------------------+
|    0 | []                          | []                              |
|    1 | [NULL]                      | [NULL]                          |
|    2 | [1, 2, 3]                   | [0, 1, 1]                       |
|    3 | [1, NULL, 3]                | [0, NULL, NULL]                 |
|    4 | [0, 1, 2, 3, NULL, 4, 6]    | [0, 1, 1, 1, NULL, NULL, 2]     |
|    5 | [1, 2, 3, 4, 5, 4, 3, 2, 1] | [0, 1, 1, 1, 1, -1, -1, -1, -1] |
|    6 | [6, 7, 8]                   | [0, 1, 1]                       |
+------+-----------------------------+---------------------------------+
```