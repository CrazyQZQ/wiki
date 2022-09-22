---
title: 组件：nacos的使用
description: nacos作为配置中心、注册中心
published: true
date: 2022-09-22T07:33:56.355Z
tags: spring cloud alibaba
editor: markdown
dateCreated: 2022-04-30T13:39:16.632Z
---

# spring cloud alibaba 组件nacos（一）

## nacos的安装（linux）

官方文档：[Nacos 快速开始](https://nacos.io/zh-cn/docs/quick-start.html)

1、下载编译后压缩包方式

```plaintext
 unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
 cd nacos/bin
```

2、启动

```plaintext
sh startup.sh -m standalone
```

3、关闭

```plaintext
sh shutdown.sh
```

4、自定义用户名和密码（默认nacos，nacos）

     1）创建数据库nacos

     2）修改配置

```plaintext
cd conf/
vim application.properties
```

     修改如下配置：

```plaintext
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=username
db.password.0=password
```

    3）修改users表数据，密码使用Bcrypt加密

## nacos用作注册中心

1、引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2、添加注解@EnableDiscoveryClient

3、配置文件（使用bootsrtap.yml）

```yml
spring:
    cloud:
        nacos:
            discovery:
                namespace: public
                password: password
                server-addr: 1.13.17.66:8848
                username: username
```

## nacos用作注册中心

1、引入依赖（需要同时引用discover）

```xml
<!--nacos-config-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2、bootstrap.yml

```yml
spring:
    cloud:
        nacos:
            config:
                contextPath: /nacos
                file-extension: yml
                password: passeord
                refresh-enabled: true
                server-addr: 1.13.17.66:8848
                username: username
                # 共享配置
                shared-configs:
                    -   dataId: share-redis.yml
                        group: DEFAULT_GROUP
                        refresh-enabled: true
                    -   dataId: share-datasource.yml
                        group: DEFAULT_GROUP
                        refresh-enabled: true
                    -   dataId: share-common.yml
                        group: DEFAULT_GROUP
                        refresh-enabled: true
                enabled: true
```

3、配置实时生效

```java
/**
 * @Description:
 * @Author QinQiang
 * @Date 2022/4/2
 **/
@RefreshScope
@Component
@Data
@ToString
public class MyConfig {
    @Value("${user.name:qwer}")
    private String name;
    @Value("${user.name:qwer}")
    private String age;
}
```