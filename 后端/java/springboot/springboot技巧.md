# springboot技巧 
**注意：优先根据[官方文档](https://springdoc.cn/spring/index.html)为主**
## 使用h2作为内置数据库,并且初始化
1.  maven依赖
```
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.2.224</version>
            <scope>runtime</scope>
        </dependency>
```
2.  yaml配置文件
```
spring:
  dataSource:  # 数据库配置
    url: jdbc:h2:mem:blog;DB_CLOSE_DELAY=-1;MODE=MySQL # 使用内存启动
  # url: jdbc:h2:file:./data/dev/blog  # 使用本地文件启动
    username: root
    password: 123456
    driverClassName: org.h2.Driver
  sql:  # 初始化数据库
    init:
      username: h2 # 用户名
      password: h2 # 密码
      mode: embedded # 加载模式，always：始终初始化数据库; embedded：内存数据库时加载; never: 不加载
      encoding: UTF-8 # 脚本种字符集解码 
      schema-locations: classpath:data/schema.sql #  优先data-locations执行，一般建表语句。
      data-locations: classpath:data/data.sql  # 一般插入数据语句。
    
```
**注意：表结构必须原生，参考以下biao'ji**
后面就可以正常使用

