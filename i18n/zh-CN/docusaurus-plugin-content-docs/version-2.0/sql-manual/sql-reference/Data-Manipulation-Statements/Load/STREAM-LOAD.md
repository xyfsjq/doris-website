---
{
    "title": "STREAM-LOAD",
    "language": "zh-CN"
}
---

## STREAM-LOAD

### Name

STREAM LOAD

## 描述

stream-load: load data to table in streaming

```
curl --location-trusted -u user:passwd [-H ""...] -T data.file -XPUT http://fe_host:http_port/api/{db}/{table}/_stream_load
```

该语句用于向指定的 table 导入数据，与普通 Load 区别是，这种导入方式是同步导入。

这种导入方式仍然能够保证一批导入任务的原子性，要么全部数据导入成功，要么全部失败。

该操作会同时更新和此 base table 相关的 rollup table 的数据。

这是一个同步操作，整个数据导入工作完成后返回给用户导入结果。

当前支持 HTTP chunked 与非 chunked 上传两种方式，对于非 chunked 方式，必须要有 Content-Length 来标示上传内容长度，这样能够保证数据的完整性。

另外，用户最好设置 Expect Header 字段内容 100-continue，这样可以在某些出错场景下避免不必要的数据传输。

参数介绍：
        用户可以通过 HTTP 的 Header 部分来传入导入参数

1. label: 一次导入的标签，相同标签的数据无法多次导入。用户可以通过指定 Label 的方式来避免一份数据重复导入的问题。
   
    当前 Doris 内部保留 30 分钟内最近成功的 label。
    
2. column_separator：用于指定导入文件中的列分隔符，默认为`\t`。如果是不可见字符，则需要加`\x`作为前缀，使用十六进制来表示分隔符。
   
    如 hive 文件的分隔符`\x01`，需要指定为`-H "column_separator:\x01"`。
    
    可以使用多个字符的组合作为列分隔符。
    
3. line_delimiter：用于指定导入文件中的换行符，默认为`\n`。可以使用做多个字符的组合作为换行符。
   
4. columns：用于指定导入文件中的列和 table 中的列的对应关系。如果源文件中的列正好对应表中的内容，那么是不需要指定这个字段的内容的。
   
    如果源文件与表 schema 不对应，那么需要这个字段进行一些数据转换。这里有两种形式 column，一种是直接对应导入文件中的字段，直接使用字段名表示；
    
    一种是衍生列，语法为 `column_name` = expression。举几个例子帮助理解。
    
    例 1: 表中有 3 个列“c1, c2, c3”，源文件中的三个列一次对应的是"c3,c2,c1"; 那么需要指定-H "columns: c3, c2, c1"
    
    例2: 表中有3个列“c1, c2, c3", 源文件中前三列依次对应，但是有多余1列；那么需要指定-H "columns: c1, c2, c3, xxx";
    
    最后一个列随意指定个名称占位即可
    
    例3: 表中有3个列“year, month, day"三个列，源文件中只有一个时间列，为”2018-06-01 01:02:03“格式；
    
    那么可以指定-H "columns: col, year = year(col), month=month(col), day=day(col)"完成导入
    
5. where: 用于抽取部分数据。用户如果有需要将不需要的数据过滤掉，那么可以通过设定这个选项来达到。
   
    例 1: 只导入大于 k1 列等于 20180601 的数据，那么可以在导入时候指定-H "where: k1 = 20180601"
    
6. max_filter_ratio：最大容忍可过滤（数据不规范等原因）的数据比例。默认零容忍。数据不规范不包括通过 where 条件过滤掉的行。

7. partitions: 用于指定这次导入所设计的 partition。如果用户能够确定数据对应的 partition，推荐指定该项。不满足这些分区的数据将被过滤掉。
   
    比如指定导入到 p1, p2 分区，-H "partitions: p1, p2"
    
8. timeout: 指定导入的超时时间。单位秒。默认是 600 秒。可设置范围为 1 秒 ~ 259200 秒。

9. strict_mode: 用户指定此次导入是否开启严格模式，默认为关闭。开启方式为 -H "strict_mode: true"。

10. timezone: 指定本次导入所使用的时区。默认为东八区。在本次导入事务中，该变量起到了替代 session variable `time_zone` 的作用。详情请见[最佳实践](#best-practice)中“涉及时区的导入”一节。

11. exec_mem_limit: 导入内存限制。默认为 2GB。单位为字节。

12. format: 指定导入数据格式，支持 csv、json、csv_with_names(支持 csv 文件行首过滤)、csv_with_names_and_types(支持 csv 文件前两行过滤)、parquet、orc，默认是 csv。

:::tip 提示
该功能自 Apache Doris  1.2 版本起支持
:::

13. jsonpaths: 导入 json 方式分为：简单模式和匹配模式。
    
    简单模式：没有设置 jsonpaths 参数即为简单模式，这种模式下要求 json 数据是对象类型，例如：
    
       ```
       {"k1":1, "k2":2, "k3":"hello"}，其中 k1，k2，k3 是列名字。
       ```
    匹配模式：用于 json 数据相对复杂，需要通过 jsonpaths 参数匹配对应的 value。
    
14. strip_outer_array: 布尔类型，为 true 表示 json 数据以数组对象开始且将数组对象中进行展平，默认值是 false。例如：
       ```
           [
            {"k1" : 1, "v1" : 2},
            {"k1" : 3, "v1" : 4}
           ]
           当strip_outer_array为true，最后导入到doris中会生成两行数据。
       ```
    
15. json_root: json_root 为合法的 jsonpath 字符串，用于指定 json document 的根节点，默认值为""。
    
16. merge_type: 数据的合并类型，一共支持三种类型 APPEND、DELETE、MERGE 其中，APPEND 是默认值，表示这批数据全部需要追加到现有数据中，DELETE 表示删除与这批数据 key 相同的所有行，MERGE 语义 需要与 DELETE 条件联合使用，表示满足 DELETE 条件的数据按照 DELETE 语义处理其余的按照 APPEND 语义处理，示例：`-H "merge_type: MERGE" -H "delete: flag=1"`

17. delete: 仅在 MERGE 下有意义，表示数据的删除条件

18. function_column.sequence_col: 只适用于 UNIQUE_KEYS，相同 key 列下，保证 value 列按照 source_sequence 列进行 REPLACE, source_sequence 可以是数据源中的列，也可以是表结构中的一列。
    
19. fuzzy_parse: 布尔类型，为 true 表示 json 将以第一行为 schema 进行解析，开启这个选项可以提高 json 导入效率，但是要求所有 json 对象的 key 的顺序和第一行一致，默认为 false，仅用于 json 格式
    
20. num_as_string: 布尔类型，为 true 表示在解析 json 数据时会将数字类型转为字符串，然后在确保不会出现精度丢失的情况下进行导入。
    
21. read_json_by_line: 布尔类型，为 true 表示支持每行读取一个 json 对象，默认值为 false。
    
22. send_batch_parallelism: 整型，用于设置发送批处理数据的并行度，如果并行度的值超过 BE 配置中的 `max_send_batch_parallelism_per_job`，那么作为协调点的 BE 将使用 `max_send_batch_parallelism_per_job` 的值。

23.hidden_columns: 用于指定导入数据中包含的隐藏列，在 Header 中不包含 columns 时生效，多个 hidden column 用逗号分割。

    ```
    hidden_columns: __DORIS_DELETE_SIGN__,__DORIS_SEQUENCE_COL__
    系统会使用用户指定的数据导入数据。在上述用例中，导入数据中最后一列数据为__DORIS_SEQUENCE_COL__。
    ```

:::tip 提示
该功能自 Apache Doris  1.2 版本起支持
:::

24. load_to_single_tablet: 布尔类型，为 true 表示支持一个任务只导入数据到对应分区的一个 tablet，默认值为 false，该参数只允许在对带有 random 分桶的 olap 表导数的时候设置。

25. compress_type: 指定文件的压缩格式。目前只支持 csv 文件的压缩。支持 gz, lzo, bz2, lz4, lzop, deflate 压缩格式。

26. trim_double_quotes: 布尔类型，默认值为 false，为 true 时表示裁剪掉 csv 文件每个字段最外层的双引号。

27. skip_lines:  整数类型，默认值为 0, 含义为跳过 csv 文件的前几行。当设置 format 设置为 `csv_with_names` 或、`csv_with_names_and_types` 时，该参数会失效。

28. comment: 字符串类型，默认值为空。给任务增加额外的信息。


:::tip 提示
该功能自 Apache Doris  1.2.3 版本起支持
:::

29. enclose:  包围符。当 csv 数据字段中含有行分隔符或列分隔符时，为防止意外截断，可指定单字节字符作为包围符起到保护作用。例如列分隔符为","，包围符为"'"，数据为"a,'b,c'",则"b,c"会被解析为一个字段。注意：当 enclose 设置为`"`时，trim_double_quotes 一定要设置为 true。
  
30. escape  转义符。用于转义在字段中出现的与包围符相同的字符。例如数据为"a,'b,'c'"，包围符为"'"，希望"b,'c 被作为一个字段解析，则需要指定单字节转义符，例如 `\`，然后将数据修改为`a,'b,\'c'`。

## 举例

1. 将本地文件'testData'中的数据导入到数据库'testDb'中'testTbl'的表，使用 Label 用于去重。指定超时时间为 100 秒
   
   ```sql
   curl --location-trusted -u root -H "label:123" -H "timeout:100" -T testData http://host:port/api/testDb/testTbl/_stream_load
   ```

2. 将本地文件'testData'中的数据导入到数据库'testDb'中'testTbl'的表，使用 Label 用于去重，并且只导入 k1 等于 20180601 的数据
        
   ```sql
   curl --location-trusted -u root -H "label:123" -H "where: k1=20180601" -T testData http://host:port/api/testDb/testTbl/_stream_load
   ```
    
3. 将本地文件'testData'中的数据导入到数据库'testDb'中'testTbl'的表，允许 20% 的错误率（用户是 defalut_cluster 中的）
        
   ```sql
   curl --location-trusted -u root -H "label:123" -H "max_filter_ratio:0.2" -T testData http://host:port/api/testDb/testTbl/_stream_load
   ```
    
4. 将本地文件'testData'中的数据导入到数据库'testDb'中'testTbl'的表，允许 20% 的错误率，并且指定文件的列名（用户是 defalut_cluster 中的）
   
   ```sql
   curl --location-trusted -u root  -H "label:123" -H "max_filter_ratio:0.2" -H "columns: k2, k1, v1" -T testData http://host:port/api/testDb/testTbl/_stream_load
   ```
    
5. 将本地文件'testData'中的数据导入到数据库'testDb'中'testTbl'的表中的 p1, p2 分区，允许 20% 的错误率。
        
   ```sql
   curl --location-trusted -u root  -H "label:123" -H "max_filter_ratio:0.2" -H "partitions: p1, p2" -T testData http://host:port/api/testDb/testTbl/_stream_load
   ```
    
6. 使用 streaming 方式导入（用户是 defalut_cluster 中的）
        
   ```sql
   seq 1 10 | awk '{OFS="\t"}{print $1, $1 * 10}' | curl --location-trusted -u root -T - http://host:port/api/testDb/testTbl/_stream_load
   ```
    
7. 导入含有 HLL 列的表，可以是表中的列或者数据中的列用于生成 HLL 列，也可使用 hll_empty 补充数据中没有的列
        
   ```sql
   curl --location-trusted -u root -H "columns: k1, k2, v1=hll_hash(k1), v2=hll_empty()" -T testData http://host:port/api/testDb/testTbl/_stream_load
   ```
    
8. 导入数据进行严格模式过滤，并设置时区为 Africa/Abidjan
        
    ```sql
    curl --location-trusted -u root -H "strict_mode: true" -H "timezone: Africa/Abidjan" -T testData http://host:port/api/testDb/testTbl/_stream_load
    ```
    
9. 导入含有 BITMAP 列的表，可以是表中的列或者数据中的列用于生成 BITMAP 列，也可以使用 bitmap_empty 填充空的 Bitmap

   ```sql
   curl --location-trusted -u root -H "columns: k1, k2, v1=to_bitmap(k1), v2=bitmap_empty()" -T testData http://host:port/api/testDb/testTbl/_stream_load
   ```
   
10. 简单模式，导入 JSON 数据
    
    表结构：

     ```sql
     `category` varchar(512) NULL COMMENT "",
     `author` varchar(512) NULL COMMENT "",
     `title` varchar(512) NULL COMMENT "",
     `price` double NULL COMMENT ""
    ```
    JSON 数据格式：
    
    ```json
    {"category":"C++","author":"avc","title":"C++ primer","price":895}
    ```
    
    导入命令：

    ```sql
    curl --location-trusted -u root  -H "label:123" -H "format: json" -T testData http://host:port/api/testDb/testTbl/_stream_load
    ```
    为了提升吞吐量，支持一次性导入多条json数据，每行为一个json对象，默认使用`\n`作为换行符，需要将`read_json_by_line`设置为true，json数据格式如下：
            
    ```json
    {"category":"C++","author":"avc","title":"C++ primer","price":89.5}
    {"category":"Java","author":"avc","title":"Effective Java","price":95}
    {"category":"Linux","author":"avc","title":"Linux kernel","price":195}
    ```
    
11. 匹配模式，导入 JSON 数据

    JSON 数据格式：

    ```json
    [
    {"category":"xuxb111","author":"1avc","title":"SayingsoftheCentury","price":895},{"category":"xuxb222","author":"2avc","title":"SayingsoftheCentury","price":895},
    {"category":"xuxb333","author":"3avc","title":"SayingsoftheCentury","price":895}
    ]
    ```
    通过指定jsonpath进行精准导入，例如只导入category、author、price三个属性
    
    ```sql
    curl --location-trusted -u root  -H "columns: category, price, author" -H "label:123" -H "format: json" -H "jsonpaths: [\"$.category\",\"$.price\",\"$.author\"]" -H "strip_outer_array: true" -T testData http://host:port/api/testDb/testTbl/_stream_load
    ```
    
    说明：
    1）如果json数据是以数组开始，并且数组中每个对象是一条记录，则需要将strip_outer_array设置成true，表示展平数组。
    2）如果json数据是以数组开始，并且数组中每个对象是一条记录，在设置jsonpath时，我们的ROOT节点实际上是数组中对象。
    
12. 用户指定 JSON 根节点

    JSON 数据格式:
    
    ```json
    {
     "RECORDS":[
    {"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},
    {"category":"22","author":"2avc","price":895,"timestamp":1589191487},
    {"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}
    ]
    }
    ```
    通过指定jsonpath进行精准导入，例如只导入category、author、price三个属性 
    
    ```sql
    curl --location-trusted -u root  -H "columns: category, price, author" -H "label:123" -H "format: json" -H "jsonpaths: [\"$.category\",\"$.price\",\"$.author\"]" -H "strip_outer_array: true" -H "json_root: $.RECORDS" -T testData http://host:port/api/testDb/testTbl/_stream_load
    ```
13. 删除与这批导入 key 相同的数据
    
    ```sql
    curl --location-trusted -u root -H "merge_type: DELETE" -T testData http://host:port/api/testDb/testTbl/_stream_load
    ```

14. 将这批数据中与 flag 列为 ture 的数据相匹配的列删除，其他行正常追加
    
    ```sql
    curl --location-trusted -u root: -H "column_separator:," -H "columns: siteid, citycode, username, pv, flag" -H "merge_type: MERGE" -H "delete: flag=1"  -T testData http://host:port/api/testDb/testTbl/_stream_load
    ```
15. 导入数据到含有 sequence 列的 UNIQUE_KEYS 表中

    ```sql
    curl --location-trusted -u root -H "columns: k1,k2,source_sequence,v1,v2" -H "function_column.sequence_col: source_sequence" -T testData http://host:port/api/testDb/testTbl/_stream_load
    ```
    
16. CSV 文件行首过滤导入
   
    文件数据：

    ```sql
    id,name,age
    1,doris,20
    2,flink,10
    ```
    通过指定`format=csv_with_names`过滤首行导入
    
    ```sql
    curl --location-trusted -u root -T test.csv  -H "label:1" -H "format:csv_with_names" -H "column_separator:," http://host:port/api/testDb/testTbl/_stream_load
    ```
17. 导入数据到表字段含有 DEFAULT CURRENT_TIMESTAMP 的表中

    表结构：
    
    ```sql
    `id` bigint(30) NOT NULL,
    `order_code` varchar(30) DEFAULT NULL COMMENT '',
    `create_time` datetimev2(3) DEFAULT CURRENT_TIMESTAMP
    ```
    
    JSON 数据格式：
    
    ```json
    {"id":1,"order_Code":"avc"}
    ```

    导入命令：

    ```sql
    curl --location-trusted -u root -T test.json -H "label:1" -H "format:json" -H 'columns: id, order_code, create_time=CURRENT_TIMESTAMP()' http://host:port/api/testDb/testTbl/_stream_load
    ```
### Keywords

    STREAM, LOAD

### Best Practice

1. 查看导入任务状态

   Stream Load 是一个同步导入过程，语句执行成功即代表数据导入成功。导入的执行结果会通过 HTTP 返回值同步返回。并以 JSON 格式展示。示例如下：

   ```json
   {
       "TxnId": 17,
       "Label": "707717c0-271a-44c5-be0b-4e71bfeacaa5",
       "Status": "Success",
       "Message": "OK",
       "NumberTotalRows": 5,
       "NumberLoadedRows": 5,
       "NumberFilteredRows": 0,
       "NumberUnselectedRows": 0,
       "LoadBytes": 28,
       "LoadTimeMs": 27,
       "BeginTxnTimeMs": 0,
       "StreamLoadPutTimeMs": 2,
       "ReadDataTimeMs": 0,
       "WriteDataTimeMs": 3,
       "CommitAndPublishTimeMs": 18
   }
   ```

   下面主要解释了 Stream load 导入结果参数：

    - TxnId：导入的事务 ID。用户可不感知。

    - Label：导入 Label。由用户指定或系统自动生成。

    - Status：导入完成状态。

        "Success"：表示导入成功。

        "Publish Timeout"：该状态也表示导入已经完成，只是数据可能会延迟可见，无需重试。

        "Label Already Exists"：Label 重复，需更换 Label。

        "Fail"：导入失败。

    - ExistingJobStatus：已存在的 Label 对应的导入作业的状态。

        这个字段只有在当 Status 为 "Label Already Exists" 时才会显示。用户可以通过这个状态，知晓已存在 Label 对应的导入作业的状态。"RUNNING" 表示作业还在执行，"FINISHED" 表示作业成功。

    - Message：导入错误信息。

    - NumberTotalRows：导入总处理的行数。

    - NumberLoadedRows：成功导入的行数。

    - NumberFilteredRows：数据质量不合格的行数。

    - NumberUnselectedRows：被 where 条件过滤的行数。

    - LoadBytes：导入的字节数。

    - LoadTimeMs：导入完成时间。单位毫秒。

    - BeginTxnTimeMs：向 Fe 请求开始一个事务所花费的时间，单位毫秒。

    - StreamLoadPutTimeMs：向 Fe 请求获取导入数据执行计划所花费的时间，单位毫秒。

    - ReadDataTimeMs：读取数据所花费的时间，单位毫秒。

    - WriteDataTimeMs：执行写入数据操作所花费的时间，单位毫秒。

    - CommitAndPublishTimeMs：向 Fe 请求提交并且发布事务所花费的时间，单位毫秒。

    - ErrorURL：如果有数据质量问题，通过访问这个 URL 查看具体错误行。

    > 注意：由于 Stream load 是同步的导入方式，所以并不会在 Doris 系统中记录导入信息，用户无法异步的通过查看导入命令看到 Stream load。使用时需监听创建导入请求的返回值获取导入结果。

2. 如何正确提交 Stream Load 作业和处理返回结果。

   Stream Load 是同步导入操作，因此用户需同步等待命令的返回结果，并根据返回结果决定下一步处理方式。

   用户首要关注的是返回结果中的 `Status` 字段。

   如果为 `Success`，则一切正常，可以进行之后的其他操作。

   如果返回结果出现大量的 `Publish Timeout`，则可能说明目前集群某些资源（如 IO）紧张导致导入的数据无法最终生效。`Publish Timeout` 状态的导入任务已经成功，无需重试，但此时建议减缓或停止新导入任务的提交，并观察集群负载情况。

   如果返回结果为 `Fail`，则说明导入失败，需根据具体原因查看问题。解决后，可以使用相同的 Label 重试。

   在某些情况下，用户的 HTTP 连接可能会异常断开导致无法获取最终的返回结果。此时可以使用相同的 Label 重新提交导入任务，重新提交的任务可能有如下结果：

   1. `Status` 状态为 `Success`，`Fail` 或者 `Publish Timeout`。此时按照正常的流程处理即可。
   2. `Status` 状态为 `Label Already Exists`。则此时需继续查看 `ExistingJobStatus` 字段。如果该字段值为 `FINISHED`，则表示这个 Label 对应的导入任务已经成功，无需在重试。如果为 `RUNNING`，则表示这个 Label 对应的导入任务依然在运行，则此时需每间隔一段时间（如 10 秒），使用相同的 Label 继续重复提交，直到 `Status` 不为 `Label Already Exists`，或者 `ExistingJobStatus` 字段值为 `FINISHED` 为止。

3. 取消导入任务

   已提交切尚未结束的导入任务可以通过 CANCEL LOAD 命令取消。取消后，已写入的数据也会回滚，不会生效。

4. Label、导入事务、多表原子性

   Doris 中所有导入任务都是原子生效的。并且在同一个导入任务中对多张表的导入也能够保证原子性。同时，Doris 还可以通过 Label 的机制来保证数据导入的不丢不重。具体说明可以参阅 [导入事务和原子性](../../../../data-operate/import/load-atomicity) 文档。

5. 列映射、衍生列和过滤

   Doris 可以在导入语句中支持非常丰富的列转换和过滤操作。支持绝大多数内置函数和 UDF。关于如何正确的使用这个功能，可参阅 [列的映射，转换与过滤](../../../../data-operate/import/load-data-convert) 文档。

6. 错误数据过滤

   Doris 的导入任务可以容忍一部分格式错误的数据。容忍率通过 `max_filter_ratio` 设置。默认为 0，即表示当有一条错误数据时，整个导入任务将会失败。如果用户希望忽略部分有问题的数据行，可以将次参数设置为 0~1 之间的数值，Doris 会自动跳过哪些数据格式不正确的行。

   关于容忍率的一些计算方式，可以参阅 [列的映射，转换与过滤](../../../../data-operate/import/load-data-convert) 文档。

7. 严格模式

   `strict_mode` 属性用于设置导入任务是否运行在严格模式下。该属性会对列映射、转换和过滤的结果产生影响，它同时也将控制部分列更新的行为。关于严格模式的具体说明，可参阅 [严格模式](../../../../data-operate/import/load-strict-mode) 文档。

8. 超时时间

   Stream Load 的默认超时时间为 10 分钟。从任务提交开始算起。如果在超时时间内没有完成，则任务会失败。

9. 数据量和任务数限制

   Stream Load 适合导入几个 GB 以内的数据，因为数据为单线程传输处理，因此导入过大的数据性能得不到保证。当有大量本地数据需要导入时，可以并行提交多个导入任务。

   Doris 同时会限制集群内同时运行的导入任务数量，通常在 10-20 个不等。之后提交的导入作业会被拒绝。

10. 涉及时区的导入

    由于 Doris 目前没有内置时区的时间类型，所有 `DATETIME` 相关类型均只表示绝对的时间点，而不包含时区信息，不因 Doris 系统时区变化而发生变化。因此，对于带时区数据的导入，我们统一的处理方式为**将其转换为特定目标时区下的数据**。在 Doris 系统中，即 session variable `time_zone` 所代表的时区。

    而在导入中，我们的目标时区通过参数 `timezone` 指定，该变量在发生时区转换、运算时区敏感函数时将会替代 session variable `time_zone`。因此，如果没有特殊情况，在导入事务中应当设定 `timezone` 与当前 Doris 集群的 `time_zone` 一致。此时意味着所有带时区的时间数据，均会发生向该时区的转换。
    例如，Doris 系统时区为 "+08:00"，导入数据中的时间列包含两条数据，分别为 "2012-01-01 01:00:00Z" 和 "2015-12-12 12:12:12-08:00"，则我们在导入时通过 `-H "timezone: +08:00"` 指定导入事务的时区后，这两条数据都会向该时区发生转换，从而得到结果 "2012-01-01 09:00:00" 和 "2015-12-13 04:12:12"。

    更详细的理解，请参阅[时区](../../../../admin-manual/cluster-management/time-zone)文档。

11. 导入的执行引擎使用

    Session Variable `enable_pipeline_load` 决定是否尝试开启 Pipeline 引擎执行 Streamload 任务。详见[导入](../../../../data-operate/import/load-manual)文档。
