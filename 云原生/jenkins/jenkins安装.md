---
title: jenkins安装
description: docker安装jenkins
published: true
date: 2022-07-29T03:29:29.747Z
tags: docker, jenkins
editor: markdown
dateCreated: 2022-07-29T03:15:34.265Z
---

# jenkins安装（docker版）
> [安装docker](http://1.13.17.66:3000/zh/%E4%BA%91%E5%8E%9F%E7%94%9F/docker/%E5%AE%89%E8%A3%85)
{.is-info}
1. 拉取镜像
```bash
docker pull jenkins/jenkins
```
2. 创建挂载目录
```bash
mkdir /home/qqWorkspace/jenkins_home
chmod 777 /home/qqWorkspace/jenkins_home
```
3. 创建容器
```bash
docker run -d -p 8090:8080 -p 8091:50000 -v /home/qqWorkspace/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins01 jenkins/jenkins
```
- `-d` 后台运行镜像
- `-p 8090:8080` 将镜像的8080端口映射到服务器的10240端口。
- `-p 8091:50000` 将镜像的50000端口映射到服务器的10241端口
- `-v /data/jenkins_home:/var/jenkins_home`挂载目录
- `-v /etc/localtime:/etc/localtime`让容器使用和服务器同样的时间设置。
- `–name jenkins01`别名
4. 登录 {$ip}:8090
![登录首页](http://124.221.239.207:9000/wiki/jenkins-login.png "jenkins-login")
复制管理员密码
```bash
cat /home/qqWorkspace/jenkins_home/secrets/initialAdminPassword
```
