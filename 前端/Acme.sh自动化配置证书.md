**注意：全部使用docker维护**
## 安装Acme
**注意：安装nginx和acme，使用[nginx多配置文件管理](./nginx.md#nginx多配置文件)**
1.编写有docker客户端的acme镜像
```Dockerfile
FROM neilpang/acme.sh:3.1.1 
RUN apk add --no-cache docker-cli
```
2.编写docker-compose.yaml

```yaml
services:
  nginx: 
    container_name: nginx
    image: nginx:1.25.4
    ports:
      - 80:80
      - 443:443
    volumes:
      - "/storage/nginx-volume/cert/:/cert" # 证书
      - "/storage/nginx-volume/workspace/:/workspace" # 项目存放地址
      - "/storage/nginx-volume/config/nginx.conf:/etc/nginx/nginx.conf"
      - "/storage/nginx-volume/config/conf.d:/etc/nginx/conf.d"
      - "/storage/nginx-volume/html:/usr/share/nginx/html" # acme验证工作文件夹
    networks:
      - local_network
  acme:
    image: acme.sh:v1
    container_name: acme
    volumes:
      - "/storage/nginx-volume/cert/:/acme.sh"  # 证书
      - "/storage/nginx-volume/html:/acme-sh-html"      # acme验证工作文件夹
      - "/var/run/docker.sock:/var/run/docker.sock"
    command: daemon           
    networks:
      - local_network
    restart: unless-stopped
```
## 配置证书源
使用**Let's Encrypt**证书源
```bash
docker exec acme acme.sh --set-default-ca --server letsencrypt
```
使用邮箱登录
```bash
docker exec acme acme.sh --register-account -m your_email@example.com
```
## 编写配置nginx验证文件
```conf
server {
    listen 80;
    server_name example.com;

    # 关键！HTTP-01 验证目录
    location /.well-known/acme-challenge {
        root /usr/share/nginx/html;
    }
}
```
## 使用acme命令生成
* 获取证书
```bash
docker exec acme --issue -d example.com --webroot /acme-sh-html
```
* 安装证书
```bash
docker exec acme --install-cert -d example.com --key-file /acme.sh/example.com.key --fullchain-file /acme.sh/example.com.crt --reloadcmd "docker exec nginx nginx -s reload"
```
## 修改nginx配置文件
```conf
server {
    listen 80;
    server_name example.com;

    location / {
        return 301 https://example.com;
    }
}

server {
   listen  443 ssl;
   server_name  example.com;

   # SSL 证书配置
   ssl_certificate /cert/example.com.crt;

   ssl_certificate_key /cert/example.com.key;

  

   # SSL 会话设置
   ssl_session_timeout 1d;                  # 超时时间建议设为 1 天
   ssl_session_cache shared:SSL:10m;       # 添加会话缓存提高性能

   # 加密套件配置（更安全的现代配置）
   ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES256-GCM-SHA384';
   ssl_prefer_server_ciphers on;
   # 协议版本（建议禁用 TLSv1.0/1.1）
   ssl_protocols TLSv1.2 TLSv1.3;          # 禁用不安全的 TLSv1.0/1.1

  

   location / {
      proxy_pass http://minio_s3; #
   }
}
```
