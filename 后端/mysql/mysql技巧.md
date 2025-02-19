# MySQL 的instr函数的使用
[参考文档](https://www.cnblogs.com/mr-wuxiansheng/p/6531221.html)
# linux MySQL 数据备份
 ```
   # centos下载安装 mysql客户端
   yum -y install mysql
   # ubuntu 下载安装 mysql客户端
   sudo apt install mysql-client
   # debian 下载mysql客户端
   apt install default-mysql-client
 ```
**编写备份脚本**
 ```bash
#!/bin/bash
# 定义MySQL数据库的连接信息

STORAGE_PATH=/storage
DB_HOST="mysql.public-server.svc.cluster.local"
DB_PORT="3306"
DB_USER="backupserver"
DB_PASS="gTz_2021"
BACKUP_DIR="public-server"
# 定义要备份的数据库列表和对应的备份目录

declare -A DATABASES=(
	["public_api"]="${STORAGE_PATH}/mysql-backup/${BACKUP_DIR}/public_api"
)

# 创建备份目录
mkdir -p ${DATABASES[@]}

# 循环遍历数据库列表并备份每个数据库
for DB_NAME in "${!DATABASES[@]}"; do
	# 定义备份文件名
	mkdir -p ${DATABASES[$DB_NAME]}
	BACKUP_FILE="${DATABASES[$DB_NAME]}/${DB_NAME}-`date +%Y-%m-%d-%H-%M-%S`.sql"
	
	# 备份MySQL数据库
	/usr/bin/mysqldump -h $DB_HOST -u $DB_USER -p$DB_PASS -P$DB_PORT $DB_NAME > $BACKUP_FILE
done
 ```
**清除文件**
```
find /storage -type f -name "*.sql" -mtime +90 -delete
```