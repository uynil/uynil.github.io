---
layout: post
title: Python Pandas ElasticSearch Tornado
tags: [Python, Pandas]
---

1. 创建对象

#### 1. using pandas from dict

``` Python
from datetime import datetime
from elasticsearch import Elasticsearch
from pandas import DataFrame, Series
import pandas as pd
import matplotlib.pyplot as plt
es = Elasticsearch(host="192.168.121.252")
res = es.search(index="_all", doc_type='logs', body={"query": {"match_all": {}}}, size=2, fields=('path','@timestamp'))

data = [{u'_id': u'a1XHMhdHQB2uV7oq6dUldg',
      u'_index': u'logstash-2014.08.07',
      u'_score': 1.0,
      u'_type': u'logs',
      u'fields': {u'@timestamp': u'2014-08-07T12:36:00.086Z',
       u'path': u'app2.log'}},
     {u'_id': u'TcBvro_1QMqF4ORC-XlAPQ',
      u'_index': u'logstash-2014.08.07',
      u'_score': 1.0,
      u'_type': u'logs',
      u'fields': {u'@timestamp': u'2014-08-07T12:36:00.200Z',
       u'path': u'app1.log'}}]
# In [35]:
df = pd.concat(map(pd.DataFrame.from_dict, data), axis=1)['fields'].T

# In [36]:
print df.reset_index(drop=True)

#                 @timestamp      path
# 0  2014-08-07T12:36:00.086Z  app2.log
# 1  2014-08-07T12:36:00.200Z  app1.log

```

#### 2. 选择对象


### 参考文章
1. [基于Django/Scrapy/ElasticSearch的搜索引擎的实现](http://www.pythontip.com/blog/post/4088/)
2. [Pandas ES Creating DataFrame From ElasticSearch Results](http://stackoverflow.com/questions/25186148/creating-dataframe-from-elasticsearch-results)
3. [github tonardo ealsticsearch](https://github.com/search?utf8=%E2%9C%93&q=elasticsearch+tornado)

