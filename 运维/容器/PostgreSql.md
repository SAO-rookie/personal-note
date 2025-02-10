# 安装Postgis数据库
```
version: "3.9"
services:
  postgis:
    image: postgis/postgis:16-3.4
    container_name: postgis
    ports:
      -  5432:5432
    command: echo "postgres" | psql -U postgres -d gisdb  -c 'CREATE EXTENSION postgis;'
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=gisdb
    volumes:
      - /storage/postgis-volumes/postgresql:/var/lib/postgresql/data
      - /storage/postgis-volumes/postgis:/var/lib/postgis/data
```
[参考文章](https://postgis.net/documentation/getting_started/)