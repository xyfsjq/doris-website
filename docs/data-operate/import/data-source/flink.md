---
{
    "title": "Flink",
    "language": "en"
}
---

Using the Flink Doris Connector, you can load data generated by Flink (such as data read from Kafka or MySQL) into Doris in real-time.

## Limitations

Requires a user-deployed Flink cluster.

## Loading Data Using Flink

For detailed steps on loading data using Flink, please refer to [Flink-Doris-Connector](../../../ecosystem/flink-doris-connector). The following steps demonstrate how to quickly load data using Flink.

### Step 1: Create Table

```sql
CREATE TABLE `students` (
  `id` INT NULL, 
  `name` VARCHAR(256) NULL,
  `age` INT NULL
) ENGINE=OLAP
UNIQUE KEY(`id`)      
COMMENT 'OLAP' 
DISTRIBUTED BY HASH(`id`) BUCKETS 1  
PROPERTIES (                                                         
"replication_allocation" = "tag.location.default: 1"
); 
```

### Step 2: Load Data Using Flink

Run bin/sql-client.sh to open the FlinkSQL console

```sql
CREATE TABLE student_sink (
    id INT,
    name STRING,
    age INT
    ) 
    WITH (
      'connector' = 'doris',
      'fenodes' = '10.16.10.6:28737',
      'table.identifier' = 'test.students',
      'username' = 'root',
      'password' = '',
      'sink.label-prefix' = 'doris_label'
);

INSERT INTO student_sink values(1,'zhangsan',123)
```

### Step 3: Verify Loaded Data

```sql
select * from test.students;                                                                                                                        
+------+----------+------+      
| id   | name     | age  |    
+------+----------+------+                                                                                                                             
|  1   | zhangsan |  123 |   
+------+----------+------+    
```