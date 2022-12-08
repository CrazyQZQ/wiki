---
title: CentOS安装jenkins
description: 
published: true
date: 2022-12-08T08:31:13.148Z
tags: 
editor: markdown
dateCreated: 2022-12-08T08:31:13.148Z
---

# CentOS安装jenkins
## 安装java
> [安装java](http://1.13.17.66:3000/zh/elk/elasticSearch/%E6%90%9C%E7%B4%A2/%E5%85%A8%E6%96%87%E6%90%9C%E7%B4%A2)
{.is-info}
## 安装jenkins
1. 安装jenkins
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
# Add required dependencies for the jenkins package
sudo yum install java-11-openjdk
sudo yum install jenkins
sudo systemctl daemon-reload
```
安装失败查看解决方案：[jenkins安装失败](https://cloud.tencent.com/developer/article/1993317)
2. 启动jenkins
```bash
sudo systemctl enable jenkins
```
```bash
sudo systemctl start jenkins
```
3. 查看jenkins状态
```bash
sudo systemctl status jenkins
```
5. 查看jenkins日志
```bash
sudo journalctl -u jenkins
```
5. 修改jenkins端口
```bash
sudo vim /usr/lib/systemd/system/jenkins.service
# 修改为想要的端口
Environment="JENKINS_PORT=8889"
# 重新加载配置文件
systemctl daemon-reload
```
6. 修改jenkins配置
```bash
vim /etc/sysconfig/jenkins
```
## 遇到的问题及解决方案
1. 解决Jenkins连接git时报错Permission denied (publickey)
原因：Jenkins创建了一个jenkins用户，并作为service以这个用户来运行。所以无论是root还是当前用户的ssh key都是不生效的
解决方案：
```bash
su root
cd /var/lib/jenkins/.ssh
ssh-keygen -t rsa -C your-email@sample.com
# 注意下一步提示保存位置的时候，要再输入
/var/lib/jenkins/.ssh/id_rsa
# Enter file in which to save the key (/root/.ssh/id_rsa): /var/lib/jenkins/.ssh/id_rsa
chown jenkins:jenkins id_rsa id_rsa.pub
cat id_rsa.pub
```
再把这个新的key添加到git系统就可以了。当然，也可以copy当前用户的key过去/var/lib/jenkins/.ssh/就好了，必须记得设置文件的owner为jenkins
2. 修改配置文件`/etc/sysconfig/jenkins`端口不生效
解决：上述步骤6，修改`/usr/lib/systemd/system/jenkins.service`文件


   