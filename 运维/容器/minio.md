# 腾讯云对象存储增加到 minio client 失败
[参考文献](https://47log.com/teng-xun-yun-dui-xiang-cun-chu-zeng-jia-dao-minio/)
**注意：虽然腾讯云遵行s3协议 但是也必须添加相关参数，链接必须去除桶名**
* example 
 https://test-1311321123.cos.ap-shanghai.myqcloud.com ==> https://cos.ap-shanghai.myqcloud.com 
```
mc alias set tc https://cos.ap-shanghai.myqcloud.com $accessKey $secretKey --api "s3v4" --path "off"
```

# 多服务器的minio集群docker-compose
```
version: '3.9'

services:
  minio:                       # 该参数 随节点服务器而改变 编写请参考extra_hosts ip和节点名称对应
    hostname: minio3            # 该参数 随节点服务器而改变 编写请参考extra_hosts 
    image: quay.io/minio/minio:latest
    command: server --console-address ":9001" http://minio{1...4}/data/{1...2}
    ports:
      - "9000:9000"
      - "9001:9001"
    extra_hosts: # 添加新节点参考如下
      - "minio1:10.0.4.5"  # 1st node
      - "minio2:10.0.4.12" # 2nd node
      - "minio3:10.0.4.8"  # 3st node
      - "minio4:10.0.4.6"  # 4st node
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=123456789
      - MINIO_DISTRIBUTED_MODE_ENABLED=yes
      - MINIO_DISTRIBUTED_NODES=minio1,minio2,minio3,minio4 # 该参数随节点名称而改变
      - MINIO_SKIP_CLIENT=yes
    volumes:
      - /stroage/1:/data/1          # 挂载文件目录
      - /stroage/2:/data/2          # 挂载文件目录
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
```

# minio 开启sftp协议
```
version: "3.9"
services:
  minio:
    container_name: minio
    image: quay.io/minio/minio:latest
    ports:
      - "9000:9000"
      - "9001:9001"
      - "8021:8021"
      - "8022:8022"
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=123456789
    volumes:
      - "/storage/data:/data"
      - "/root/.ssh/id_rsa:/opt/id_rsa"
    command: server /data --console-address ":9001" --ftp="address=:8021" --sftp="address=:8022" --sftp="ssh-private-key=/opt/id_rsa"
```