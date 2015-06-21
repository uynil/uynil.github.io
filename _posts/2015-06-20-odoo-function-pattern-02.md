---
layout: post
title: oodo 设计模式一 - 消息通知
---
## 问题
系统根据单据更新发送消息通知相关人员。

举例：
生产订单下发，库管人员就会得到需要发货的消息。

相关的设计模式有：
* 重写动作函数
* 自动化动作
* 定制开发，根据单据状态，发送消息


下面就，讨论不同模式的实现方法，和利弊。

## 重写动作函数

odoo v8 中 确认生产订单 通过workflow完成，
workflow 调用的确认函数是:

```
def action_confirm(self, cr, uid, ids, context=None):
        """ Confirms production order.
        @return: Newly generated Shipment Id.
        """
        user_lang = self.pool.get('res.users').browse(cr, uid, [uid]).partner_id.lang
        context = dict(context, lang=user_lang)
        uncompute_ids = filter(lambda x: x, [not x.product_lines and x.id or False for x in self.browse(cr, uid, ids, context=context)])
        self.action_compute(cr, uid, uncompute_ids, context=context)
        for production in self.browse(cr, uid, ids, context=context):
            self._make_production_produce_line(cr, uid, production, context=context)

            stock_moves = []
            for line in production.product_lines:
                if line.product_id.type != 'service':
                    stock_move_id = self._make_production_consume_line(cr, uid, line, context=context)
                    stock_moves.append(stock_move_id)
                else:
                    self._make_service_procurement(cr, uid, line, context=context)
            if stock_moves:
                self.pool.get('stock.move').action_confirm(cr, uid, stock_moves, context=context)
            production.write({'state': 'confirmed'})
        return 0
```

重写函数, 增加以下代码实现。 

```
SUPER(mrp_prodduction, self).action_confirm(cr, uid, ids, context=None)
self.message_post(cr, uid, ids, body=_("Order %s confirmed. Please Send Material") % self._description, context=context)

```

修改结果
![rewrite_function.png](http://upload-images.jianshu.io/upload_images/72534-0a21de9a15b13b48.png)


优点: 简单直接
缺点: 需要找到代码函数，重写函数。

## 定制，根据状态变化
此方法需要修改代码，在单据状态变化的时候，自动推送消息。

依赖代码部分
模块集成

```
    _inherit = ['mail.thread', 'ir.needaction_mixin']
```

定义状态, 增加track_visibility属性

```
'state': fields.selection(
            [('draft', 'New'), ('cancel', 'Cancelled'), ('confirmed', 'Awaiting Raw Materials'),
                ('ready', 'Ready to Produce'), ('in_production', 'Production Started'), ('done', 'Done')],
            string='Status', readonly=True,
            track_visibility='onchange', copy=False,
```
定义_trace 字段以及参数
ps. mrp.production 中未定义_track, 故状态更新 不会推送消息通知。
```
# Automatic logging system if mail installed
    # _track = {
    #   'field': {
    #       'module.subtype_xml': lambda self, cr, uid, obj, context=None: obj[state] == done,
    #       'module.subtype_xml2': lambda self, cr, uid, obj, context=None: obj[state] != done,
    #   },
    #   'field2': {
    #       ...
    #   },
    # }
    # where
    #   :param string field: field name
    #   :param module.subtype_xml: xml_id of a mail.message.subtype (i.e. mail.mt_comment)
    #   :param obj: is a browse_record
    #   :param function lambda: returns whether the tracking should record using this subtype
```
其中 module.subtype_xml 需要在xml中定义消息类型。 例如 account_voucher 的跟踪消息类型
```
 <!-- Voucher-related subtypes for messaging / Chatter -->
        <record id="mt_voucher_state_change" model="mail.message.subtype">
            <field name="name">Status Change</field>
            <field name="res_model">account.voucher</field>
            <field name="default" eval="False"/>
            <field name="description">Status changed</field>
        </record>
```

优点：根据状态或其他字段自动推送消息。
缺点：定义复杂。

## 自动化动作
创建自动话动作，定义对象和条件
![Automatci_action.png](http://upload-images.jianshu.io/upload_images/72534-0651084743bbbbbe.png)

定义动作: 更改负责人 或增加关注者(本例中可以增加仓库人员)
![set_action1.png](http://upload-images.jianshu.io/upload_images/72534-8a302e7bae83a86b.png)

或 更复杂动作，用服务器动作定义
![create_server_action.png](http://upload-images.jianshu.io/upload_images/72534-fa8d719d6b047bf3.png)

优点 : 用户可配置
缺点: server action 需要写python代码

#总结

以上三种方法，都是使用message_post方法发送消息给关注者，如需使用其他发送消息方法，需要在mail thread寻找新的方法。
方法三，可以自定义配置条件，也可以增加关注者，也可以增加复杂动作，灵活。
方法一，对开发者来说更直接。



