
# linux postgresql 数据备份
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