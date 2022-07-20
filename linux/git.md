---
title: linux 使用git
description: 
published: true
date: 2022-07-20T02:26:13.397Z
tags: 
editor: markdown
dateCreated: 2022-07-13T12:05:47.172Z
---

# 安装

```plaintext
yum -y install git
git --version
git config --global user.name "Your Name"
git config --global user.email "yourEmail@yourDomain.com"


在命令行创建公密钥，一路回车即可
ssh-keygen -t rsa
cat /root/.ssh/id_rsa.pub	#公钥，将以下内容加入 github 的 key

```