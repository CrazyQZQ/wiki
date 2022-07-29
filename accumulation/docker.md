---
title: docker安装
description: docker的使用（linux）
published: true
date: 2022-07-29T02:59:51.742Z
tags: docker
editor: markdown
dateCreated: 2022-05-07T11:33:49.182Z
---

# 安装docker，本机环境：CentOS8

参考：https://docs.docker.com/engine/install/centos/

1. 删除旧版本

```
yum remove docker \
            docker-client \
            docker-client-latest \
            docker-common \
            docker-latest \
            docker-latest-logrotate \
            docker-logrotate \
            docker-engine
```

2. 安装软件包（提供实用程序）并设置稳定存储库。

```
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

3. 安装最新版本的 Docker 引擎、容器化和 Docker Compose

```
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

4. 启动 Docker。

```
systemctl start docker
```

安装报错：
`problem with installed package podman-1.6.4-10.module_el8.2.0+305+5e198a41.x86_64`
输入一下命令继续安装：
```bash
yum install --allowerasing docker-ce
```

# 常用命令

参考：[Docker 命令大全 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-command-manual.html)

```
// 拉取镜像
docker pull
// 查看镜像
docker images
// 启动容器
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
// 在容器 mynginx 中以交互模式执行容器内 /root/runoob.sh 脚本
docker exec -it mynginx /bin/sh
```

# 安装docker-compose

参考：https://blog.csdn.net/AlexanderRon/article/details/123412922

1. 从git上拉取docker-compose，注意：这里版本我安装的是`1.28.2`，如果需要安装对应某个版本请更改。

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2. 授权docker-compose

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

3. 建立软连接

```plaintext
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

4. 测试

```
docker-compose --version
```