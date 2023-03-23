---
title: windows安装mysql8绿色版
description: windows安装mysql8绿色版
published: true
date: 2023-03-23T07:23:03.507Z
tags: 
editor: markdown
dateCreated: 2023-02-15T12:17:31.672Z
---

# windows安装mysql8绿色版

## 8.0下载

https://dev.mysql.com/downloads/file/?id=509736

## 卸载旧版（绿色版）

- 删注册表 regedit （不存在的就不管）
    HKEY_LOCAL_MACHINE/SYSTEM/ControlSet001/Services/Eventlog/Application/MySQL
    HKEY_LOCAL_MACHINE/SYSTEM/ControlSet002/Services/Eventlog/Application/MySQL
    HKEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/Services/Eventlog/Application/MySQL
- 删除服务
   管理员身份运行CMD.EXE，然后输入

```java
sc delete MySQL
```

- 删除安装文件夹

## 安装MySql8 绿色版

注意：不要新建data目录

- 官网下载社区服务器版本，解压，进入到解压目录，新建my.ini文件，输入以下内容：

```java
[mysqld]
#设置端口号
port=3306
#设置mysql的安装目录 报错就改成双斜杠 \\
basedir=D:\DevTools\mysql-8.0.18-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\DevTools\mysql-8.0.18-winx64\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```

- 右键点击windows左下放大镜图标，搜索框输入cmd，弹出框选择选择“以管理员身份运行”管理员身份启动CMD.exe进入MySql8 的bin目录，依次输入以下命令：

```bash
mysqld install 
mysqld  --initialize-insecure --console
```

执行命令 `mysqld --initialize --console`，这个命令执行完会初始化一个root的密码，显示到控制台，也可以直接执行 `mysqld --initialize` 无法看到随机密码，**记住这个随机密码**，第一次登录会使用到。

```bash
net start mysql
mysql -uroot -p
user mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY  '123456';
```

## 忘记初始密码

1. 以系统管理员身份运行cmd.

2. 查看mysql是否已经启动，如果已经启动，就停止：net stop mysql.

3. 切换到MySQL安装路径下：D:\Program Files\mysql-5.6.41-winx64\mysql-5.6.41-winx64；如果已经配了环境变量，可以不用切换了。

4. 在命令行输入：

   ```bash
   #作用是绕过登录权限
   mysqld -nt --skip-grant-tables
   ```

   

5. 以管理员身份重新启动一个cmd命令窗口，输入：

   ```bash
   mysql -uroot -p
   ```

6. 如果要修改密码的话，在命令行下 依次 执行下面的语句

   ```bash
   use mysql;
   #这里改为你要设置的密码
   update user set password=password("new_pass") where user="root";// 'new_pass' 
   #刷新权限，使修改生效
   flush privileges;   
   quit;
   ```

7. 重新启动MYSQL

   ```bash
   netstat -start mysql
   ```

   