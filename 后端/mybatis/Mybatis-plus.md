# Mybatis-plus
## 逻辑删除
参考：[官方文档](https://baomidou.com/guides/logic-delete/)
### 1.时间类型
使用时间类型作为逻辑删除,以下配置要同时使用
yaml配置
```yaml
mybatis-plus:  
  global-config:  
    db-config:  
      logic-delete-field: deleteTime  
      logic-delete-value: "now()"  
      logic-not-delete-value: 'null'
```
代码配置
```java
@TableLogic(value = "null", delval = "now()")  
private OffsetDateTime deleteTime;
```
### 2.整数类型
使用整数类型作为逻辑删除
```yaml
mybatis-plus:  
  global-config:  
    db-config:  
	  logic-delete-field: delete_status
	  logic-delete-value: 1
	  logic-not-delete-value: 0
```