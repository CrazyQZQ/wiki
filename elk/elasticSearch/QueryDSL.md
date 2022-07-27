---
title: Elastic Search Query Dsl
description: Elastic Search查询语句
published: true
date: 2022-07-27T12:20:04.646Z
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
### 查找多个精确值（terms）
```
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "terms" : { 
                    "price" : [20, 30]
                }
            }
        }
    }
}
```
*解释* 查找价格字段值包含 20 或 30
`term`和`terms`是 包含（contains） 操作，而非 等值（equals）。
如果我们有一个 term（词项）过滤器 { "term" : { "tags" : "search" } } ，它会与以下两个文档 同时 匹配：
```
{ "tags" : ["search"] }
{ "tags" : ["search", "open_source"] }
```
尽管第二个文档包含除 search 以外的其他词，它还是被匹配并作为结果返回。
### 范围（range）
+ `gt`: `>`大于（greater than）
+ `lt`: `<` 小于（less than）
+ `gte`: `>=` 大于或等于（greater than or equal to）
+ `lte`: `<=` 小于或等于（less than or equal to）
如果想要范围无界（比方说 >20 ），只须省略其中一边的限制
#### 日期范围
1. 直接使用字符串比较
```
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-07 00:00:00"
    }
}
```
2. 日期计算
+ 这个过滤器会一直查找时间戳在过去一个小时内的所有文档，让过滤器作为一个时间 *滑动窗口（sliding window）* 来过滤文档。
```
"range" : {
    "timestamp" : {
        "gt" : "now-1h"
    }
}
```
+ 日期计算还可以被应用到某个具体的时间，并非只能是一个像 now 这样的占位符。只要在某个日期后加上一个双管符号 (`||`) 并紧跟一个日期数学表达式就能做到：
```
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-01 00:00:00||+1M" 
    }
}
```
#### 字符串范围
`range`查询同样可以处理字符串字段，字符串范围可采用 *字典顺序（lexicographically）* 或 *字母顺序（alphabetically）* 。例如，下面这些字符串是采用字典序（lexicographically）排序的：

+ 5, 50, 6, B, C, a, ab, abb, abc, b

如果我们想查找从`a`到`b`（不包含）的字符串，同样可以使用 `range` 查询语法：
```
"range" : {
    "title" : {
        "gte" : "a",
        "lt" :  "b"
    }
}
```

