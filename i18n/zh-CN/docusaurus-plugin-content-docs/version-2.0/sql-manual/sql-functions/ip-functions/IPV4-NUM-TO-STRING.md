---
{
"title": "IPV4_NUM_TO_STRING",
"language": "zh-CN"
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

## IPV4_NUM_TO_STRING

## 描述

IPV4_NUM_TO_STRING

## 语法

`VARCHAR IPV4_NUM_TO_STRING(BIGINT ipv4_num)`

接受一个类型为 Int16、Int32、Int64 且大端表示的 IPv4 的地址，返回相应 IPv4 的字符串表现形式，格式为 A.B.C.D（以点分割的十进制数字）。

## 注意事项

`对于负数或超过4294967295 （即 '255.255.255.255'）的入参都返回NULL，表示无效收入`

## 举例

```
mysql> select ipv4_num_to_string(3232235521);
+--------------------------------+
| ipv4_num_to_string(3232235521) |
+--------------------------------+
| 192.168.0.1                    |
+--------------------------------+
1 row in set (0.01 sec)

mysql> select num,ipv4_num_to_string(num) from ipv4_bi;
+------------+---------------------------+
| num        | ipv4_num_to_string(`num`) |
+------------+---------------------------+
|         -1 | NULL                      |
|          0 | 0.0.0.0                   |
| 2130706433 | 127.0.0.1                 |
| 4294967295 | 255.255.255.255           |
| 4294967296 | NULL                      |
+------------+---------------------------+
7 rows in set (0.01 sec)
```

### keywords

IPV4_NUM_TO_STRING, IP
