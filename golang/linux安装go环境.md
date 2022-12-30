---
title: linux安装go环境
description: 
published: true
date: 2022-12-30T01:15:37.701Z
tags: 
editor: markdown
dateCreated: 2022-12-30T01:15:37.701Z
---

# 安装
1. 下载go的安装包
Golang官网下载地址：[https://golang.org/dl/](https://golang.org/dl/)
```bash
按照包版本：go1.11.5.linux-amd64.tar.gz：
下载地址：https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz
```
2. 将安装包解压放到到usr/local中，并解压
```bash
cd /usr/local
mkdir go
cd go
wget https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz
tar -zxvf go1.11.5.linux-amd64.tar.gz
```
3. 将/usr/local/go/bin添加到环境变量中
```bash
vim /etc/profile
# 在最后一行添加
export GOROOT=/usr/local/go/go
export PATH=$PATH:$GOROOT/bin
# 保存退出后source一下（vim 的使用方法可以自己搜索一下）
source /etc/profile
```
4. 执行go -version查看是否安装成功