---
{
    "title": "HDFS",
    "language": "zh-CN"
}
---

## HDFS

### Name

<version since="1.2">

hdfs

</version>

## 描述

HDFS表函数（table-valued-function,tvf），可以让用户像访问关系表格式数据一样，读取并访问 HDFS 上的文件内容。目前支持`csv/csv_with_names/csv_with_names_and_types/json/parquet/orc`文件格式。

## 语法
```sql
hdfs(
  "uri" = "..",
  "fs.defaultFS" = "...",
  "hadoop.username" = "...",
  "format" = "csv",
  "keyn" = "valuen" 
  ...
  );
```

**参数说明**

访问hdfs相关参数：
- `uri`：（必填） 访问hdfs的uri。
- `fs.defaultFS`：（必填）
- `hadoop.username`： （必填）可以是任意字符串，但不能为空
- `hadoop.security.authentication`：（选填）
- `hadoop.username`：（选填）
- `hadoop.kerberos.principal`：（选填）
- `hadoop.kerberos.keytab`：（选填）
- `dfs.client.read.shortcircuit`：（选填）
- `dfs.domain.socket.path`：（选填）

文件格式相关参数
- `format`：(必填) 目前支持 `csv/csv_with_names/csv_with_names_and_types/json/parquet/orc`
- `column_separator`：(选填) 列分割符, 默认为`,`。 
- `line_delimiter`：(选填) 行分割符，默认为`\n`。

    下面6个参数是用于json格式的导入，具体使用方法可以参照：[Json Load](../../../data-operate/import/file-format/json)

- `read_json_by_line`： (选填) 默认为 `"true"`
- `strip_outer_array`： (选填) 默认为 `"false"`
- `json_root`： (选填) 默认为空
- `json_paths`： (选填) 默认为空
- `num_as_string`： (选填) 默认为 `false`
- `fuzzy_parse`： (选填) 默认为 `false`

    <version since="dev">下面2个参数是用于csv格式的导入</version>

- `trim_double_quotes`： 布尔类型，选填，默认值为 `false`，为 `true` 时表示裁剪掉 csv 文件每个字段最外层的双引号
- `skip_lines`： 整数类型，选填，默认值为0，含义为跳过csv文件的前几行。当设置format设置为 `csv_with_names` 或 `csv_with_names_and_types` 时，该参数会失效 

## 举例s

读取并访问 HDFS 存储上的csv格式文件
```sql
MySQL [(none)]> select * from hdfs(
            "uri" = "hdfs://127.0.0.1:842/user/doris/csv_format_test/student.csv",
            "fs.defaultFS" = "hdfs://127.0.0.1:8424",
            "hadoop.username" = "doris",
            "format" = "csv");
+------+---------+------+
| c1   | c2      | c3   |
+------+---------+------+
| 1    | alice   | 18   |
| 2    | bob     | 20   |
| 3    | jack    | 24   |
| 4    | jackson | 19   |
| 5    | liming  | 18   |
+------+---------+------+
```

可以配合`desc function`使用

```sql
MySQL [(none)]> desc function hdfs(
            "uri" = "hdfs://127.0.0.1:8424/user/doris/csv_format_test/student_with_names.csv",
            "fs.defaultFS" = "hdfs://127.0.0.1:8424",
            "hadoop.username" = "doris",
            "format" = "csv_with_names");
```

### Keywords

    hdfs, table-valued-function, tvf

### Best Practice

  关于HDFS tvf的更详细使用方法可以参照 [S3](./s3.md) tvf, 唯一不同的是访问存储系统的方式不一样。
