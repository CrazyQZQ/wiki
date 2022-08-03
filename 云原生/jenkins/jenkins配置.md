---
title: jenkins配置
description: jenkins配置及构建
published: true
date: 2022-08-03T01:54:36.442Z
tags: jenkins
editor: markdown
dateCreated: 2022-08-03T01:54:36.442Z
---

# Jenkins配置

## 全局配置（Global Tool Configuration）
Manager Jenkins > Global Tool Configuration
配置jdk，maven等

## 配置SSH远程发布

1. 安装插件Publish Over SSH （Manage Jenkins > Plugin Manager）
2. 进入系统设置配置远程ssh地址 （Manage Jenkins > Configure System）
![新增ssh](http://124.221.239.207:9000/wiki/jenkins_ssh_setting.png)

## 新建项目
1. 新增item （Dashboard > 新建item）
2. 构建配置
![新增ssh](http://124.221.239.207:9000/wiki/item_setting.jpeg)
注意：*源码管理 git* 需要使用私钥
```bash
cat /root/.ssh/is_rsa
```


