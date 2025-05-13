# nginx多配置文件
*注意：以docker为标准*
1. 基本配置文件
```
worker_processes 1;
events {
    worker_connections 1024;
}
http {
    include mime.types;
    default_type application/octet-stream;

    sendfile on;
    keepalive_timeout 65;
    client_max_body_size 2048m;

    include /etc/nginx/conf.d/*.conf;  # 你其他配置文件的地址
}

```
2. 其他配置文件
**注意：配置文件必须以.conf结尾**
```
# 内容模板
server{
    listen 80;
    server_name localhost;
    location / {
        root /usr/local/nginx/html;
        index index.html index.htm;
    }
}
```
3. dockercompose脚本
```
version: '3.9'
services:
  nginx: 
    container_name: nginx
    image: harbor.changtech.cn/common/base-nginx:v1
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - "/storage/nginx-volume/cert/:/cert"
      - "/storage/nginx-volume/workspace/:/workspace"
      - "/storage/nginx-volume/config/nginx.conf:/etc/nginx/nginx.conf"
      - "/storage/nginx-volume/config/conf.d:/etc/nginx/conf.d"
```
# nginx 双端配置
```
    server {
        listen       80;
        server_name  localhost;
        location / {
            # 检查User-Agent字符串，判断是否为移动设备
            if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
                # 如果是移动设备，则重定向到移动端的网站
                rewrite ^ http://127.0.0.1$request_uri? permanent;
                break;
            }
            # 如果不是移动设备，则将请求代理到PC端的后端服务器
            proxy_pass http://127.0.0.1/;
        }
    }
```

# nginx 开启gzip
```
# 开启gzip
gzip on;

# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1k;

# gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明
gzip_comp_level 2;

# 进行压缩的文件类型。javascript有多种形式。其中的值可以在 mime.types 文件中找到。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;

# 是否在http header中添加Vary: Accept-Encoding，建议开启
gzip_vary on;

# 禁用IE 6 gzip
gzip_disable "MSIE [1-6]\.";
```
# nginx配置https
```
server {
     listen  443 ssl;
     server_name  office.hello.cn;

     # SSL 证书配置
     ssl_certificate /cert/office-view/cert.pem;
     ssl_certificate_key /cert/office-view/cert.key;

  

	 # SSL 会话设置

     ssl_session_timeout 1d;                  # 超时时间建议设为 1 天

     ssl_session_cache shared:SSL:10m;       # 添加会话缓存提高性能

  

     # 加密套件配置（更安全的现代配置）
     ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES256-GCM-SHA384';

     ssl_prefer_server_ciphers on;

  

     # 协议版本（建议禁用 TLSv1.0/1.1）

     ssl_protocols TLSv1.2 TLSv1.3;          # 禁用不安全的 TLSv1.0/1.1

  
     location / {
         proxy_pass  http://172.17.0.1:8080;
     }
}

  

server {
     listen 80;
     server_name office.hello.cn;
     return 301 https://office.hello.cn;
}
```
# Vue使用HTML5模式 nginx配置
```
worker_processes  1;


events {
  worker_connections  1024;
}


http {
  include       mime.types;
  default_type  application/octet-stream;
  
  sendfile        on;
  keepalive_timeout  65;
  
  server {
    listen       80;
    server_name  localhost;
  
  
    location / {
      root   /opt/dist/;             
      try_files $uri $uri/ /index.html;  
      index index.html index.htm;
    }
  }
}
```