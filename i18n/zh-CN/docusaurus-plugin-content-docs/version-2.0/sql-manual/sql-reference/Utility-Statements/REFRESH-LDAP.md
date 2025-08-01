---
{
    "title": "REFRESH-LDAP",
    "language": "zh-CN"
}
---

## REFRESH-LDAP

### Name



REFRESH-LDAP




## 描述

该语句用于刷新 Doris 中 LDAP 的缓存信息。修改 LDAP 服务中用户信息或者修改 Doris 中 LDAP 用户组对应的 role 权限，可能因为缓存的原因不会立即生效，可通过该语句刷新缓存。Doris 中 LDAP 信息缓存默认时间为 12 小时，可以通过 `ADMIN SHOW FRONTEND CONFIG LIKE 'ldap_user_cache_timeout_s';` 查看。

语法：

```sql
REFRESH LDAP ALL;
REFRESH LDAP [for user_name];
```

## 举例

1. 刷新所有 LDAP 用户缓存信息

    ```sql
    REFRESH LDAP ALL;
    ```

2. 刷新当前 LDAP 用户的缓存信息

    ```sql
    REFRESH LDAP;
    ```

3. 刷新指定 LDAP 用户 user1 的缓存信息

    ```sql
    REFRESH LDAP for user1;
    ```

### Keywords

REFRESH, LDAP

### Best Practice

