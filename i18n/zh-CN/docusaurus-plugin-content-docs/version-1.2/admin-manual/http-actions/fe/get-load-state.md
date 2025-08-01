---
{
    "title": "Get Load State",
    "language": "zh-CN"
}
---

# Get Load State

## Request

`GET /api/<db>/get_load_state`

## Description

返回指定label的导入事务的状态
执行完毕后，会以Json格式返回这次导入的相关内容。当前包括以下字段：
	Label：本次导入的 label，如果没有指定，则为一个 uuid
	Status：此命令是否成功执行，Success表示成功执行
	Message： 具体的执行信息
	State: 只有在Status为Success时才有意义
		UNKNOWN: 没有找到对应的Label
		PREPARE: 对应的事务已经prepare，但尚未提交
		COMMITTED: 事务已经提交，不能被cancel
		VISIBLE: 事务提交，并且数据可见，不能被cancel
		ABORTED: 事务已经被ROLLBACK，导入已经失败
    
## Path parameters

* `<db>`

    指定数据库

## Query parameters

* `label`

    指定导入label

## Request body

无

## Response

```
{
	"msg": "success",
	"code": 0,
	"data": "VISIBLE",
	"count": 0
}
```

如label不存在，则返回：

```
{
	"msg": "success",
	"code": 0,
	"data": "UNKNOWN",
	"count": 0
}
```
    
## Examples

1. 获取指定label的导入事务的状态。

    ```
    GET /api/example_db/get_load_state?label=my_label
    
    {
    	"msg": "success",
    	"code": 0,
    	"data": "VISIBLE",
    	"count": 0
    }
    ```
