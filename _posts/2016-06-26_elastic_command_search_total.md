---
layout: post
title: Python Pandas ElasticSearch Tornado
tags: [Python, Pandas]
---
  简介
  概念
  安装部署



#### 1. 简介

在关系型数据库中，我们常常要查询所有满足条件的数据。

elastic search 为了避免高cpu，高内存的消耗，查询都会有两个参数 size, from。


#### 查询计数
返回值 his.total 就是总数。
``` bash
    '查询文档'
    curl 'yimian.office:9200/shakespeare/_search?q=*&pretty'
    curl -XPOST 'yimian.office:9200/shakespeare/_search?pretty' -d '
    {
      "query": { "match_all": {} }
    }'
    '需要注意的是，一旦查询结果返回，查询过程完全结束，服务器不会保留任何结果在服务器端。'
    '''''''''

    curl -XPOST 'yimian.office:9200/bank/_search?pretty' -d '
    {
      "query": { "match_all": {} },
      "_source": ["account_number", "balance"],
      "from": 10,
      "size": 10,
      "sort": { "balance": { "order": "desc" } }
    }'
```

#### 用scroll 循环查询

To fetch all the records you can also use scroll concept.. It's like cursor in db's..
If you use scroll, you can get the docs batch by batch.. It will reduce high cpu usage and also memory usage..

``` bash
curl -XGET 'localhost:9200/twitter/tweet/_search?scroll=1m' -d '
{
   "query": {
    "match" : {
        "title" : "elasticsearch"
     }
 }
}'
```


##### filters aggs

{
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : {
          "errors" :   { "term" : { "body" : "error"   }},
          "warnings" : { "term" : { "body" : "warning" }}
        }
      },
      "aggs" : {
        "monthly" : {
          "histogram" : {
            "field" : "timestamp",
            "interval" : "1M"
          }
        }
      }
    }
  }
}

### Reference
1. [Elastic Search](http://stackoverflow.com/questions/22405341/elastic-search-size-to-unlimited)