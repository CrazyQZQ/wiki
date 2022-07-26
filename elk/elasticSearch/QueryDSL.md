---
title: Elastic Search Query Dsl
description: Elastic Search查询语句
published: true
date: 2022-07-26T12:06:31.726Z
tags: 
editor: markdown
dateCreated: 2022-07-26T12:06:31.726Z
---

# 查询和过滤
> [官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html){.isInfo}
## 查询和筛选上下文
```console
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```
+ `title`必须包含`Search`
+ `content`必须包含`Elasticsearch`
+ 在查询结果中过滤出：`status`包含完整`published`（不分词）
+ 在查询结果中过滤出：`publish_date`大于`2015-01-01`

### 布尔（bool）查询
包含`must`, `should`, `must_not`, `filter`几种子查询
+ `must`：子句（查询）必须出现在匹配的文档中，并有助于分数。
+ `should`：子句（查询）应出现在匹配的文档中（也可以不出现），并有助于分数。
+ `must_not`：子句（查询）不得出现在匹配的文档中。子句在筛选器上下文中执行，这意味着将忽略评分，并考虑对子句进行缓存。由于忽略评分，因此将返回所有文档的分数为0
+ `filter`：基于`must`字句，不会影响分数。

### 权重（Boosting）查询
```console
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "text": "apple"
        }
      },
      "negative": {
        "term": {
          "text": "pie"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```
*解释*：搜索包含apple，尽量不包含pie的doc，如果包含了pie，不会说排除掉这个doc，而是说将这个doc的分数降低
+ `positive`：（必需，查询对象）要运行的查询，任何返回的文档都必须与此查询匹配，用于提高匹配文档的***相关性分数***的查询。
+ `negative`：（必需，查询对象）要运行的查询，用于降低匹配文档的***相关性分数***的查询。
+ `negative_boost`：（必需，查询对象）,包含了`negative` term的doc，分数乘以`negative_boost`，分数降低
### 常量分数查询（Constant score query）
```console
GET /_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": { "user.id": "kimchy" }
      },
      "boost": 1.2
    }
  }
}
```
*解释*：包装筛选查询，并返回*相关性分数*等于参数值的每个匹配文档。
