---
{
    "title": "ARRAY_LAST",
    "language": "zh-CN"
}
---

## 描述

使用一个 lambda 表达式和一个 ARRAY 作为输入参数，lambda 表达式为布尔型，用于对 ARRAY 中的每个元素进行判断返回值。
返回数组中的最后一个 lambda(arr1[i]) 值不为 0 的元素。当数组中所有元素进行 lambda(arr1[i]) 都为 0 时，结果返回`NULL`值。

## 语法

```sql
ARRAY_LAST(<lambda>, <arr>)
```

## 参数

| 参数 | 说明 | 
| --- |---|
| `<lambda>` | lambda 表达式，表达式中输入的参数为 1 个或多个，必须和后面的输入 array 列数量一致。在 lambda 中可以执行合法的标量函数，不支持聚合函数等。 |
| `<arr>` | ARRAY 数组     |

## 返回值

返回值最后不为 0 的索引。如果没找到满足此条件的索引，则返回 NULL。

## 举例

```sql
select array_last(x->x>2, [1,2,3,0]) ;
```

```text
+------------------------------------------------------------------------------------------------+
| array_last(array_filter(ARRAY(1, 2, 3, 0), array_map([x] -> x(0) > 2, ARRAY(1, 2, 3, 0))), -1) |
+------------------------------------------------------------------------------------------------+
|                                                                                              3 |
+------------------------------------------------------------------------------------------------+
```

```sql
select array_last(x->x>4, [1,2,3,0]) ; 
```

```text
+------------------------------------------------------------------------------------------------+
| array_last(array_filter(ARRAY(1, 2, 3, 0), array_map([x] -> x(0) > 4, ARRAY(1, 2, 3, 0))), -1) |
+------------------------------------------------------------------------------------------------+
|                                                                                           NULL |
+------------------------------------------------------------------------------------------------+
```
