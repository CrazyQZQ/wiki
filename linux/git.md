---
title: linux 使用git
description: 
published: true
date: 2022-07-20T02:30:18.525Z
tags: 
editor: markdown
dateCreated: 2022-07-13T12:05:47.172Z
---

# 安装
1. 安装
```plaintext
yum -y install git
git --version
```
2. 配置
```
git config --global user.name "Your Name"
git config --global user.email "yourEmail@yourDomain.com"
```

3. 在命令行创建公密钥，一路回车即可
```
ssh-keygen -t rsa
```
4. 将公钥内容加入 github 的 key
```
cat /root/.ssh/id_rsa.pub
```
`settings` -> `SSH and GPG keys` -> `New SSH key`