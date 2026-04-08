
# postgreSQL 
## springboot
### 1.postgreSQL时区类型
*注意：当使用表语句使用 TIMESTAMPTZ 类型时，java的时间类型要选择 OffsetDateTime，不要选择其他类型，否则会类型转化错误。*
```java
public class FileInfo { 
	// pattern: 定义时间格式 
	// timezone: 极其重要！对于 OffsetDateTime，这确保了 JSON 序列化时的时区转换 
	@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8") 
	private OffsetDateTime createTime; 
}
```
### 2.postgreSQL的连接配置
```
jdbc:postgresql://192.168.31.3:5432/apartment-project?directConnection=true&stringtype=unspecified&userTimezone=Asia/Shanghai&reWriteBatchedInserts=true
```

| **参数**                           | **作用说明**                                                                                                               |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **`directConnection=true`**      | **强制直连**。它要求驱动程序不要尝试去探测集群拓扑（比如询问谁是主库）。在连接 **PgBouncer**、**连接池代理** 或 **特定从库** 时，这能防止驱动程序因为无法识别代理背后的真实节点而报错。             |
| **`stringtype=unspecified`**     | **极其重要（针对兼容性）**。它告诉驱动：如果 Java 传了一个 String，但数据库列类型不匹配，不要报错，让数据库自己去猜（隐式转换）。这通常是为了解决 **JSON/JSONB** 或 **UUID** 类型匹配失败的问题。 |
| **`userTimezone=Asia/Shanghai`** | **时区对齐**。确保 JDBC 驱动在处理 `TIMESTAMP` 或你正在用的 `TIMESTAMPTZ` 时，强制使用北京时间，避免产生 8 小时的偏移误差。                                     |
| **`reWriteBatchedInserts=true`** | PostgreSQL JDBC 驱动中**性能优化**的“核武器”。开启它之后，MyBatis 或 Hibernate 的批量插入效率通常能提升 **5 到 10 倍**。                                 |

## linux postgreSQL 数据备份
 ```bash
# debian 下载 postgresql 客户端
apt install postgresql-client -y
 ```
 **备份脚本**
```bash
#!/bin/bash
export PGPASSWORD=gTz_8702
  
STORAGE_PATH=/storage
DB_HOST="127.0.0.1"
DB_USER="root"
DB_PASSWORD="123456"

declare -A DATABASES=(
	["public_service_map"]="${STORAGE_PATH}/postgresql-backup/public_service_map"
)

mkdir -p ${DATABASES[@]}

# 循环遍历数据库列表并备份每个数据库
for DB_NAME in "${!DATABASES[@]}"; do

	# 定义备份文件名
	mkdir -p ${DATABASES[$DB_NAME]}
	BACKUP_FILE="${DATABASES[$DB_NAME]}/${DB_NAME}-`date +%Y-%m-%d-%H-%M-%S`.sql"
	
	# 备份MySQL数据库
	pg_dump -h $DB_HOST -p 5432 -U $DB_USER $DB_NAME -F p -f $BACKUP_FILE

done
```