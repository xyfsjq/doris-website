---
{
    "title": "FROM_BASE64",
    "language": "zh-CN"
}
---

## 描述

返回对输入的字符串进行 Base64 解码后的结果。特殊情况：

- 当输入字符串不正确时（出现非 Base64 编码后可能出现的字符串）将会返回 NULL

## 语法

```sql
FROM_BASE64 ( <str> )
```

## 参数

| 参数      | 说明              |
|---------|-----------------|
| `<str>` | 需要被 Base64 解码的字符串 |

## 返回值

参数 <str> 通过 Base64 解码后的结果。特殊情况：

- 当输入字符串不正确时（出现非 Base64 编码后可能出现的字符串）将会返回 NULL

## 举例

```sql
SELECT FROM_BASE64('MQ=='),FROM_BASE64('MjM0'),FROM_BASE64(NULL)
```

```text
+---------------------+---------------------+-------------------+
| from_base64('MQ==') | from_base64('MjM0') | from_base64(NULL) |
+---------------------+---------------------+-------------------+
| 1                   | 234                 | NULL              |
+---------------------+---------------------+-------------------+
```