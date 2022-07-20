---
title: elasticsearch安装
description: elasticsearch安装
published: true
date: 2022-07-20T03:05:29.376Z
tags: 
editor: markdown
dateCreated: 2022-05-07T11:40:44.139Z
---

# docker安装启动es

1. 为 Elasticsearch 和 Kibana 创建一个新的 Docker 网络

```
docker network create elastic
```

2. 在 Docker 中启动 Elasticsearch。将为用户生成密码并输出到终端，以及用于注册 Kibana 的注册令牌。（docker logs 查看日志输出，找到密码）

```
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.1.3
```

3. 重置密码

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

4. 将安全证书从 Docker 容器复制到本地计算机。http\_ca.crt

```
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt /home/qqWorkSpace/elk/elasticsearch/
```

5. 打开一个新终端，并使用从 Docker 容器复制的文件，通过进行经过身份验证的调用来验证是否可以连接到 Elasticsearch 集群。出现提示时，输入用户的密码。

```
curl --cacert /home/qqWorkSpace/elk/elasticsearch/http_ca.crt -u elastic https://localhost:9200
```

## 安装kibana

1. 拉取镜像

```
docker pull docker.elastic.co/kibana/kibana:8.1.3
```

2. 启动容器

```
docker run --name kib01 --net elastic -p 5601:5601 -it docker.elastic.co/kibana/kibana:8.1.3
```

# es的配置

Elasticsearch有三个配置文件：

- elasticsearch.yml用于配置 Elasticsearch
- jvm.options用于配置 Elasticsearch JVM 设置
- log4j2.properties用于配置 Elasticsearch 日志记录

1. 进入容器内部：

```
docker exec -it 6f0e7f387863 /bin/bash
```

2. 进入文件夹 usr/share/elasticsearch/config 可以看到配置文件

```
cd /usr/share/elasticsearch/config
```

3. 使用如下可以把修改后的文件复制进容器

```
docker cp /home/qqWorkSpace/elk/elasticsearch/elasticsearch.yml es01:/usr/share/elasticsearch/config/
```

# 使用docker-elk

## 使用github上docker镜像
### 准备：安装docker
> [安装docker](/zh/accumulation/docker)
1. 下载后使用如下命令（需要安装docker和docker-compose）

```
>cd docker-elk
>docker-compose up -d
```

2. .env中可以为内置用户配置密码

3. 配置logstash 管道

```
>cd docker-elk/logstash/pipeline
```

4. 管道配置示例：logstash.conf

```
input {
#	beats {
#		port => 5044
#	}

#	tcp {
#		port => 5000
#	}
    redis {
        batch_count => 1
        data_type => "list"
        key => "logs"
        host => "1.13.17.66"
        port => 6379
        password => "qq123456"
        threads => 5
    }
}

## Add your filters / logstash plugins configuration here

output {
	elasticsearch {
		hosts => "elasticsearch:9200"
        index => "logs-%{+YYYY.MM.dd}"
		user => "logstash_internal"
		password => "${LOGSTASH_INTERNAL_PASSWORD}"
	}
}
```

## 安装IK分词器

1. 下载对应es版本：[Releases · medcl/elasticsearch-analysis-ik (github.com)](https://github.com/medcl/elasticsearch-analysis-ik/releases)

2. 进入容器内部，mkdir /usr/share/elasticsearch/plugins/ik

docker cp /tmp/elasticsearch-analysis-ik-8.1.3.zip elasticsearch:/usr/share/elasticsearch/plugins/ik

3. 重启es

```plaintext
curl -XPOST http://localhost:9200/index/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            }
        }

}'
```