---
layout: post
title: odoo设计模式
---

odoo 提供了一套开发框架和一系列好用的ERP模块。基于此，我们会在odoo框架odoo提供的模块基础上解决一系列公司管理上的信息化问题。而一个同样问题可以用几种不同的odoo框架提供的方法和odoo模块的功能解决。这种odoo框架的修改方法和odoo标准功能的组合，我称之为odoo设计模式。好的设计模式，可以大大的程序效率，也可以提高用户体验。

下文，我就以两个设计模式和一个具体问题，初步谈谈我对odoo的设计模式解决自动更新odoo数据类问题的看法。
希望可以抛砖引玉。

## 设计模式一 -- 手工更新+定时任务批量更新

代码片段

定义功能

```
  @api.model
    def scheduler_update_orders_draft(self):
        draft_orders = self.search([('state','in',['draft'])])
        draft_orders._compute_order_draft()
        return

```

定义按钮

```
  <button name="scheduler_update_orders_draft" string='Update Order Draft' />

```

定义cron

```
<openerp>
    <data>
        <record id="scheduler_check_order_draft_job" model="ir.cron">
            <field name="name">Check Draft Order</field>
            <field name="model">sale.order</field>
            <field name="function">scheduler_update_orders_draft</field>
            <field name="args">([])</field>
            <field name="interval_number">2</field>
            <field name="interval_type">hours</field>
            <field name="priority">10</field>
            <field name="active">1</field>  
        </record>
    </data>
</openerp>
```

odoo 使用举例：

* cron 举例: 删除wazard临时保存数据；检查并执行自动化动作；

* 手工更新举例: 订单上更新税和total，执行button_dummy；所有wizard类计算；warehouse scheduler 计算。


## 设计模式二 -- function 函数自动更新数据 

代码片段

```
is_draft = fields.Boolean(string='Draft Order', compute='_compute_order_draft', store=True)

@api.depends('shipping_arrival_date', 'default_delay')
def _compute_order_draft(self):
    for rec in self:
        order_delay = self._get_lines_delay() # get max of lines delay
        self.is_draft = False
        if not self.shipping_arrival_date:
            continue
        shipping_arrival_date = datetime.strptime(self.shipping_arrival_date, '%Y-%m-%d').date()

        cur_order_lead_time = (shipping_arrival_date - date.today()).days
        if  cur_order_lead_time > order_delay:
            self.is_draft = True
    return
```

根据其他字段('shipping_arrival_date', 'default_delay')的更改,更新当前字段is_draft的值

备注：

* store=False, 计算字段不存储。读取字段，显示字段的时候，进行计算。举例， 界面上加载字段，或程序中读取字段。优势是，字段肯定是最新的值。要注意的是，为了避免加载界面时，太慢，界面上应尽量避免store=False的function字段。
* store=True, 计算字段会进行存储。 更改条件满足的时候，进行计算，并存储在数据库上。有点是，读取快。缺点是，其他字段更改时会做大量自动话。

odoo 使用举例：

* 产品的可用库存, 虚拟库存,  product_onhand, product_virtual, 这两个字段是store=False的虚拟库存。每次计算时，会计算所有相关的stock move 和stock quant；而产品页面tree 视图上有显示。另外，因为store 为False, 产品无法根据这两个字段进行搜索和过滤。
如果store = True, 每次stock move 的更改就会需要重新计算相关产品的这两个字段，数据量大了，stock move的完成会很慢。

* Sale Order 的 amount_total, amount_subtotal, amount_tax

## 问题举例 - 计算 产品 的交期 lead time

问题描述：

产品的交期-lead time, 系统需要计算出目前产品的交期，并把交期反馈给系统客户。

可以用上述两种模式计算交期，两种模式取舍如下:
* 手工计算+ 半夜批量。 
	缺点： 交期数据会不准确, 会有延时性。
	优点： 保证系统的流畅性，保证系统性能。手工操作也可以提供人为控制的可能性。

* function字段自动计算产品交期。

	缺点：数据量大的时候，会导致系统相关操作时间增加(会驱动function字段的计算)。
	优点： 自动化。


## 结论：
这两种方案需要做性能和数据准确性的取舍，当我们对性能要求更高的时候，牺牲数据的准确性，可以选择手工+定时方案；需要数据准确性，而对性能要求不高时，可以选择自动化方案。


## 改进点
手工更新 + 半夜批量更新 

可以改进为

手工更新 + 半夜批量更新 + 小批量定时更新 (记录 待更新对象队列，定时更新队列中的对象)

## [寰享科技(深圳) 正在招聘 初级/高级odoo 技术工程师][job_link]
[job_link]: http://simple-is-better.com/jobs/866 "Eilco Shenzhen hire odoo developers"

如果有任何疑问，欢迎发邮件到 lin.yu#elico-corp.com 跟我讨论。（# 改成@）
