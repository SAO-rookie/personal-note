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
# Mysql 主从同步
[shardingSphere proxy中间件文档](https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-proxy/)
## 数据库配置文件
1.主数据库配置文件
```
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

character-set-server=utf8mb4

collation-server = utf8mb4_unicode_ci

init-connect='SET NAMES utf8mb4'

init_connect='SET collation_connection = utf8mb4_unicode_ci'

sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'

# 服务id是唯一的,不能重复
server-id=100
log-bin=mysql-bin

# 主库忽视同步某些数据库
binlog_ignore_db = mysql
binlog_ignore_db = sys
binlog_ignore_db = information_schema
binlog_ignore_db = performance_schema
```
2.从节点数据库配置文件
```
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

character-set-server=utf8mb4

collation-server = utf8mb4_unicode_ci

init-connect='SET NAMES utf8mb4'

init_connect='SET collation_connection = utf8mb4_unicode_ci'

sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'

# 服务id是唯一的,不能重复
server_id=101 
log-bin=mysql-slave-bin 
relay_log=edu-mysql-relay-bin

# 从库忽视同步某些数据库
replicate-ignore-db = mysql
replicate-ignore-db = sys
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
```
##  数据库同步脚本
主节点脚本
**注意：主节点脚本，创建同步用户时记得限制ip访问**
```
#!/bin/bash

set -e

ROOT="root"
ROOT_PASSWORD="123456"

# 创建同步用户时记得限制ip访问
CREATE_SYNC_USER_SQL="USE mysql; CREATE USER 'slave'@'192.168.0.%' IDENTIFIED BY '123456';"
LIMIT_SYNC_USER_ACCESS_IP="USE mysql; update user set host='192.168.0.%' where user='slave';"
GRANT_SYNC_PRIVILEGES_SQL="USE mysql; GRANT ALL PRIVILEGES ON *.* TO 'slave'@'192.168.0.%' WITH GRANT OPTION;"
UPDATE_ENCRYPT_METHOD="USE mysql; ALTER USER 'slave'@'192.168.0.%' IDENTIFIED WITH mysql_native_password BY '123456';"
REFRESH_DATA="USE mysql; flush privileges;"

mysql -u$ROOT -p$ROOT_PASSWORD -e "$CREATE_SYNC_USER_SQL"

mysql -u$ROOT -p$ROOT_PASSWORD -e "$LIMIT_SYNC_USER_ACCESS_IP"

mysql -u$ROOT -p$ROOT_PASSWORD -e "$GRANT_SYNC_PRIVILEGES_SQL"

mysql -u$ROOT -p$ROOT_PASSWORD -e "$UPDATE_ENCRYPT_METHOD"

mysql -u$ROOT -p$ROOT_PASSWORD -e "$REFRESH_DATA"

history -c
```
从节点脚本
```
#!/bin/bash

set -e

MASTER_HOST="192.168.0.100"
MASTER_PORT="3306"
MASTER_SYNC_USER="slave"
MASTER_SYNC_USER_PASSWORD="123456"

sleep 10

RESULT=`mysql -h$MASTER_HOST -P$MASTER_PORT -u$MASTER_SYNC_USER -p$MASTER_SYNC_USER_PASSWORD -e "SHOW MASTER STATUS;" | grep -v grep |tail -n +2| awk '{print $1,$2}'`
LOG_FILE_NAME=`echo $RESULT | grep -v grep | awk '{print $1}'`
LOG_FILE_POS=`echo $RESULT | grep -v grep | awk '{print $2}'`

SLAVE_USER=root
SLAVE_PASSWORD=gTz_2021

SYNC_SQL="""CHANGE MASTER TO MASTER_HOST='$MASTER_HOST',MASTER_PORT=$MASTER_PORT,MASTER_USER='$MASTER_SYNC_USER',MASTER_PASSWORD='$MASTER_SYNC_USER_PASSWORD',MASTER_LOG_FILE='$LOG_FILE_NAME',MASTER_LOG_POS=$LOG_FILE_POS,MASTER_CONNECT_RETRY=30;"""

START_SYNC_SQL="START REPLICA;"
STATUS_SQL="SHOW REPLICA STATUS\G;"

mysql -u$SLAVE_USER -p$SLAVE_PASSWORD -e "$SYNC_SQL"

mysql -u$SLAVE_USER -p$SLAVE_PASSWORD -e "$START_SYNC_SQL"

mysql -u$SLAVE_USER -p$SLAVE_PASSWORD -e "$STATUS_SQL"

history -c
```
## docker-compose 脚本
```
version: '3.9'

services:
  mysql-master:
    container_name: mysql-master
    image: mysql:8.0.30
    ports:
      - "8000:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=gTz_2021
      - TZ=Asia/Shanghai
    volumes:
      - "/storage/mysql-master-one-volume/config/my.cnf:/etc/mysql/my.cnf"
      - "/storage/mysql-master-one-volume/script/mysql-init.sh:/docker-entrypoint-initdb.d/master.sh" # 主节点脚本
    restart: always
    networks:
      shardingSphere:
        ipv4_address: 192.168.0.100
  mysql-slave-one:
    container_name: mysql-slave-one
    image: mysql:8.0.30
    ports:
      - "8001:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=gTz_2021
      - TZ=Asia/Shanghai
    volumes:
      - "/storage/mysql-slave-one-volume/config/my.cnf:/etc/mysql/my.cnf"
      - "/storage/mysql-slave-one-volume/script/mysql-init.sh:/docker-entrypoint-initdb.d/slave.sh"  # 从节点脚本
    restart: always
    networks:
      shardingSphere:
        ipv4_address: 192.168.0.101
    depends_on:
      - mysql-master
networks:
  shardingSphere:
    ipam:
      driver: default
      config:
        - subnet: "192.168.0.0/24"
```

## 双主模式

需要两个数据库都为主库，并且互相做数据备份，**注意：两个数据库**

基本配置和主库一样，但是需要添加以下配置

```
log-slave-updates
auto-increment-increment=2
auto-increment-offset=1 # 这个参数 在一号数据库填写1 在一号数据库填写2

# 从库忽视同步某些数据库
replicate-ignore-db = mysql
replicate-ignore-db = sys
replicate-ignore-db = information_schema 
replicate-ignore-db = performance_schema
```

然后在不同的数据库执行以下sql

```
change master to 
master_host='172.16.0.13', # 你要备份服务器ip
master_port = 3307,
master_user='slave',
master_password='123456',
master_log_file='mysql-bin.000003', # 你要备份的数据库日志文件
master_log_pos=1152, # 你要备份的数据库pos
master_connect_retry = 30;  
```