**注意：全部使用docker维护**
## 安装
**注意：安装nginx和acme，使用[nginx多配置文件管理](./nginx/# nginx多配置文件)**


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
      - "/storage/nginx-volume/cert/:/acme.sh"  
      - "/storage/nginx-volume/html:/acme-sh-html"      
      - "/var/run/docker.sock:/var/run/docker.sock"
    command: daemon           
    networks:
      - local_network
    restart: unless-stopped
```