---
title: centOS8安装使用code-server
description: centOS8安装使用code-server
published: true
date: 2022-07-20T11:32:33.115Z
tags: 
editor: markdown
dateCreated: 2022-07-20T11:31:57.124Z
---

# CentOS8安装使用code-server
## 安装
[官方文档](https://coder.com/docs/code-server/latest/install#fedora-centos-rhel-suse)
### 本地使用rpm安装
1. 下载安装包，[链接](https://github.com/coder/code-server/releases)
2. 安装,密码在`~/.config/code-server/config.yaml`里，默认使用8080端口
```bash
sudo rpm -i code-server-$VERSION-amd64.rpm
sudo systemctl enable --now code-server@$USER
# Now visit http://127.0.0.1:8080. Your password is in ~/.config/code-server/config.yaml
```
3. 外网访问：使用nginx代理
部分nginx.conf
```conf
location /code-server/ {
    proxy_pass http://localhost:8080/;
    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection upgrade;
    proxy_set_header Accept-Encoding gzip;
}
```
4. 访问 http://${ip}/code-server/
## 升级
1. 下载安装包
2. 安装rpm
```bash
rpm -ivh code-server-$VERSION-amd64.rpm
systemctl enable --now code-server@$USER
```
