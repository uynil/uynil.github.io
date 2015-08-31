---
layout: post
title: odoo 性能调优以及负载均衡
tags: [odoo]
---
## 1 Odoo负载均衡

### 1.1 Odoo服务器负载均衡

* Nginx, odoo服务器做负载均衡，空间换时间
* odoo使用多线程模式
![](http://upload-images.jianshu.io/upload_images/72534-188b1b5f61f15e8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.2数据库负载均衡，读写分离

使用postgres_XC或pg_pool进行postgres负载均衡

## 2 数据库性能调优

* 数据库使用物理机
* postgres参数调优,如共享内存
http://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server

## 3性能度量以及监控

### 3.1监控

使用监控工具（munin, cacti, newrelic）度量服务器 cpu，内存，硬盘数据。

3.2 数据库分析

* 检查pg数据库，pg_stat_activity, pg_locks, pg_statio_user_tables等数据
* 使用pg分析工具  以及 EXPLAIN ANALYZE检查sql查询效率
* 分离数据库和odoo附件

4 Odoo定制模块性能调优

* Tree View，尽量使用分页，而不是提高每页显示条数。

来自odoo官方的建议：

* Stored computed fields触发太多：   增加触发条件限制，避免无谓存储。
* 避免在主数据 (product, location, user, company）上增加计算类字段
* 搜索 Domain不合理 -多表搜索，效率非常低
    - 举例 ([('sale _id.order_lines.product_id ','!=', False)])
* 业务逻辑重写在 create(), write()函数中。性能会降低    因为这些函数会被反复高频调用。 
    - 避免重写 sale order line, stock move的         这些函数 避免在主数据 (product,       location, user, company）的这些函数
* 误用批量(Batch)API - browse, write 等函数已经支持batch
* 手动锁表