**注意：全部使用docker维护**
## 安装Acme
**注意：安装nginx和acme，使用[nginx多配置文件管理](./nginx.md#nginx多配置文件)**

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
    image: neilpang/acme.sh:3.1.1
    container_name: acme
    volumes:
      - "/storage/nginx-volume/cert/:/acme.sh"  # 证书
      - "/storage/nginx-volume/html:/acme-sh-html"      # acme验证工作文件夹
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
```
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
docker exec acme --install-cert -d example.com --key-file /acme.sh/example.com.key --fullchain-file /acme.sh/example.com.crt
```
## 修改nginx配置文件
