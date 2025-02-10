# Mysql配置
## mysq8.x 自定义配置

```
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# 下面的配置看不懂 使用Deepseek帮你分析

max_connections=1000

character-set-server=utf8mb4

collation-server = utf8mb4_unicode_ci

init-connect='SET NAMES utf8mb4'

init_connect='SET collation_connection = utf8mb4_unicode_ci'

skip-character-set-client-handshake

sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION' 

```

# linux MySQL 数据备份
 ```
   # centos下载安装 mysql客户端
   yum -y install mysql
   # ubuntu 下载安装 mysql客户端
   sudo apt install mysql-client
 ```
**编写定时备份脚本**
 ```
# 定义MySQL数据库的连接信息
DB_HOST="127.0.0.1"
DB_PORT="3306"
DB_USER="backupserver"
DB_PASS="gTz_2021"
# 备份地址
BACKUP_DIR="public-server"

# 定义要备份的数据库列表和对应的备份目录
declare -A DATABASES=(
    ["public_api"]="/opt/mysql-backup/${BACKUP_DIR}/data/public_api"
    ["public_file"]="/opt/mysql-backup/${BACKUP_DIR}/data/public_file"
    ["public_secrets"]="/opt/mysql-backup/${BACKUP_DIR}/data/public_secrets"
    ["public_security"]="/opt/mysql-backup/${BACKUP_DIR}/data/public_security"
)

# 创建备份目录
mkdir -p ${DATABASES[@]}

# 循环遍历数据库列表并备份每个数据库
for DB_NAME in "${!DATABASES[@]}"; do
    # 定义备份文件名
    BACKUP_FILE="${DATABASES[$DB_NAME]}/${DB_NAME}-`date +%Y-%m-%d-%H-%M-%S`.sql"

    # 备份MySQL数据库
    /usr/bin/mysqldump -h $DB_HOST -u $DB_USER -p$DB_PASS -P$DB_PORT  $DB_NAME > $BACKUP_FILE
done

find /opt/mysql-backup/${BACKUP_DIR}/data  -type f -name "*.sql" -mtime +90 -delete
 ```
 

# Mysql 添加新用户

```
CREATE USER 'slave'@'%' IDENTIFIED BY '123456'; # 创建从用户
update user set host='%' where user='slave'; # 修改从用户 无ip访问限制
## 权限的粒度要更细 请问询chatgpt
GRANT ALL PRIVILEGES ON *.* TO 'slave'@'%' WITH GRANT OPTION; # 授予所有权限给用户在所有数据库上：
ALTER USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY '123456';# 修改加密方式
flush privileges; # 刷新数据权限
```