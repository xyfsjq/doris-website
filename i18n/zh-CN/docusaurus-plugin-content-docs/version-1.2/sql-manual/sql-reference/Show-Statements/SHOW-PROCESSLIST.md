---
{
    "title": "SHOW-PROCESSLIST",
    "language": "zh-CN"
}
---

## SHOW-PROCESSLIST

### Name

SHOW PROCESSLIST

## 描述

显示用户正在运行的线程，需要注意的是，除了 root 用户能看到所有正在运行的线程外，其他用户都只能看到自己正在运行的线程，看不到其它用户正在运行的线程

语法：

```sql
SHOW [FULL] PROCESSLIST
```

说明：

- Id: 就是这个线程的唯一标识，当我们发现这个线程有问题的时候，可以通过 kill 命令，加上这个Id值将这个线程杀掉。前面我们说了show processlist 显示的信息时来自information_schema.processlist 表，所以这个Id就是这个表的主键。
- User: 就是指启动这个线程的用户。
- Host: 记录了发送请求的客户端的 IP 和 端口号。通过这些信息在排查问题的时候，我们可以定位到是哪个客户端的哪个进程发送的请求。
- Cluster：集群名称
- Db: 当前执行的命令是在哪一个数据库上。如果没有指定数据库，则该值为 NULL 。
- Command: 是指此刻该线程正在执行的命令。
- Time: 表示该线程处于当前状态的时间。
- State: 线程的状态，和 Command 对应。
- Info: 一般记录的是线程执行的语句。默认只显示前100个字符，也就是你看到的语句可能是截断了的，要看全部信息，需要使用 show full processlist。

常见的 Command 类型如下：

- Query: 该线程正在执行一个语句
- Sleep: 正在等待客户端向它发送执行语句
- Quit: 该线程正在退出
- Kill : 正在执行 kill 语句，杀死指定线程

其他类型可以参考 [MySQL 官网解释](https://dev.mysql.com/doc/refman/5.6/en/thread-commands.html)

## 举例

1. 查看当前用户正在运行的线程
   ```SQL
   SHOW PROCESSLIST
   ```

### Keywords

    SHOW, PROCESSLIST

### Best Practice

