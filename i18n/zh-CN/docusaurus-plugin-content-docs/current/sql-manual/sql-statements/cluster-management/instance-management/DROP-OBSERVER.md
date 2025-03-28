---
{
    "title": "DROP OBSERVER",
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





## 描述

该语句是删除 FRONTEND 的 OBSERVER 角色的节点，（仅管理员使用！）

语法：

```sql
ALTER SYSTEM DROP OBSERVER "follower_host:edit_log_port"
```

说明：

1. host 可以是主机名或者 ip 地址
2. edit_log_port : edit_log_port 在其配置文件 fe.conf

## 示例

1. 添加一个 FOLLOWER 节点

   ```sql
   ALTER SYSTEM DROP OBSERVER "host_ip:9010"
   ```

## 关键词

    ALTER, SYSTEM, DROP, OBSERVER, ALTER SYSTEM

### 最佳实践

