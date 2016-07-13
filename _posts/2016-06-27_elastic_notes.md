---
layout: post
title: Python Pandas ElasticSearch Tornado
tags: [Python, Pandas]
---
  简介
  概念
  安装部署



#### 1. 简介

Elastic Search 支持相当复杂的搜索情形。像“有水果有肉有主食红黄白相间成粒状”这种可以把 MySQL 的检索虐得死去活来的查询，Elastic Search 可以轻松告诉你这应该是名菜“哈密瓜年糕牛肉粒”。下面看看最简单却很实用的搜索：查询所有值。

#### 排序
1. 排序
2. 分组排序

"aggs" : {
        "order_by_title" : {
            "terms" : {
              "field" : "title",
              "order": {
                "_term" : "asc" 
              }
            }
        }
    }

3. 分组取数
aggs = {
                term1: {
                    'terms': {
                        'field': 'forum.brand.name',
                    },
                    'aggs': {
                        'line_stats': {
                          'stats': {'field': 'nlp.rate'}
                        }
                    }
                }
            }

https://www.elastic.co/guide/en/elasticsearch/guide/current/_intrinsic_sorts.html

### Reference
1. [Elastic Search](http://stackoverflow.com/questions/22405341/elastic-search-size-to-unlimited)
2. [kibana 语法](http://ju.outofmemory.cn/entry/228662)