---
title: jdk安装
description: 
published: true
date: 2022-12-07T10:57:08.156Z
tags: 
editor: markdown
dateCreated: 2022-12-07T10:57:08.156Z
---

# centOS安装jdk
## 卸载卸载已安装的jdk
1. 查询已安装的jdk
```bash
[root@10-60-74-87 ~]# yum list installed |grep java
java-1.8.0-openjdk-headless.x86_64            1:1.8.0.265.b01-4.el8                             @AppStream       
javapackages-filesystem.noarch                5.3.0-1.module_el8.0.0+11+5b8c10bd                @AppStream       
tzdata-java.noarch                            2020d-1.el8  
```
2. 卸载jdk
```bash
yum -y remove java-1.8.0-openjdk-headless.x86_64
```
## 安装jdk
1. 查看所有jdk版本
```bash
yum search java | grep -i --color JDK
```
2. 安装指定版本JDK
```bash
yum install java-17-openjdk-devel.x86_64
```
3. 配置环境变量
  + 查看JAVA_HOME安装目录
    ```bash
    dirname $(readlink $(readlink $(which java)))
    /usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64/bin
    ```
  + 配置profile文件
    ```bash
    vim /etc/profile
    ```
    ```bash
    export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64
    export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/jre/lib/dt.jar:$JAVA_HOME/lib/tool.jar
    export PATH=$PATH:$JAVA_HOME
    ```
  + 使配置文件生效
    ```bash
    source /etc/profile
    ```
4. 查看jdk版本
```bash
[root@10-60-74-87 ~]# java -version
openjdk version "17.0.1" 2021-10-19 LTS
OpenJDK Runtime Environment 21.9 (build 17.0.1+12-LTS)
OpenJDK 64-Bit Server VM 21.9 (build 17.0.1+12-LTS, mixed mode, sharing)
```

    
  
  