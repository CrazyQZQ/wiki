---
title: mysql踩坑
description: mysql使用时遇到的问题
published: true
date: 2022-07-20T02:53:39.545Z
tags: mysql
editor: markdown
dateCreated: 2022-04-30T14:09:44.906Z
---

# 创建用户，修改密码

## 1、关闭密码模式

```
cd /etc/my.cnf.d/
vim mysql-server.cnf
```

增加如下配置

```
skip-grant-tables
```

## 2、进入mysql 

```plaintext
mysql -u root
```

**创建用户：**

```plaintext
create user '#userName'@'#host' identified by '#passWord';
```

#userName 代表你要创建的此数据库的新用户账号

#host 代表访问权限，如下

%代表通配所有host地址权限(可远程访问)

localhost为本地权限(不可远程访问）

**查看用户：**

```
user mysql;
select host, user, authentication_string, plugin from user;
```

**用户授权：**

```
grant #auth on #databaseName.#table to '#userName'@'#host';
```

#auth 代表权限，如下

all privileges 全部权限

select 查询权限

select,insert,update,delete 增删改查权限

select,\[…\]增…等权限

#databaseName 代表数据库名

#table 代表具体表，如下

\*代表全部表

A,B 代表具体A,B表

#userName 代表用户名

#host 代表访问权限，如下

%代表通配所有host地址权限(可远程访问)

localhost为本地权限(不可远程访问)

指定特殊Ip访问权限 如10.138.106.102

**切记一定要刷新授权才可生效：**

```
flush privileges;
```

**查看用户权限：**

```
show grants for '#userName'@'#host';
```