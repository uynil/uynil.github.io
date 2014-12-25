---
layout: blog
title: odoo qweb 报表开发简易教程
---

#{{page.title}}

### {{ page.date | date_to_string }}

正文之前， 先说两件事：
一. 强调下odoo (原 openerp) 是开源软件, 源码是最好的老师，关于如何开发qweb报表，请多看源码。
例子，point_of_sale模块有多个qweb报表的例子。
二. 贴个招聘链接。
## [Elico Corp (深圳) 正要招聘odoo技术工程师][job_link]
[job_link]: http://simple-is-better.com/jobs/866 "Eilco Shenzhen hire odoo developers"

# qweb report 介绍
openerp 7版 使用 webkit 和 rml 报表引擎。 6版 用 rml。
qweb 是8版采用的新报表引擎，webkit and rml 已在8版中弃用。

qweb 也是odoo web服务器的网页渲染引擎。也就是说，8版中，odoo统一中网页模板和报表模块的渲染技术。

## 本文结构
* qweb
* 如何创建一个qweb报表

## qweb
基本上 , 

1. 程序员在 xml 代码中编写 动作，报表，视图，样式，在python 代码中编写
2. 模块安装，上面编写中的动作，报表，视图会存在数据库中。
3. 当用户点击视图上的按钮，或打开一个新网页，它会驱动动作，调用 网页 javascript API. 网页 javascript 脚本会 
    * 找到对应的视图id 和数据库中的视图代码，渲染到网页上。
    * 找到对应的数据id和数据库中的值，渲染到网页上。

4. 打应报表时，odoo 使用 wkhtmltopdf 把网页样式转换到pdf格式.

### qweb 语法介绍
* 数据
	`t-field`, `t-esc`
 
* 循环, 条件

	```
	<p t-foreach="[1, 2, 3]" t-as="i">
    	<t t-esc="i"/>
	</p>
	```
	```
	<t t-if="condition">
        <p>ok</p>
    </t>
	```

## 如何创建一个qweb报表
### 0. 模块结构
```
	| report
		|	customize_report.py
	| views
		|	report_layout_view.xml
	| report.xml
	| __init__.py
	| __openerp__.py
	| ...
```
### 1. 创建一个 report

* if no 2nd step, the value of file and name  2nd step.
* if 2nd step, the value of  should be the template id in 2nd step

```
<report 
            id="report_sale_order_libiya_xxx"
            string="Sale Order Libiya"
            model="sale.order" 
            report_type="qweb-pdf"
            file="module.report_sale_order_xxx" 
name="module.report_sale_order_xxx" 
        />
```
### 2. 创建一个可翻译的报表记录 (可选)
```
<template id="report_sale_order_xxx">
    <t t-call="report.html_container">
        <t t-foreach="doc_ids" t-as="doc_id">
            <t t-raw="translate_doc(doc_id, doc_model, 'partner_id.lang', 'module.report_sale_order_xxx_document')"/>
        </t>
    </t>
</template>
```

### 创建报表样式

odoo 使用 bootstrap 作为网页样式：
http://www.w3cschool.cc/bootstrap/bootstrap-grid-system.html


```
<template id="report_sale_order_xxx_document">
    <t t-call="report.external_layout">
    <div class="page">
        <div class="oe_structure"/>
        <table class="dest_address">
        <tr>
            <td>
                <strong>Customer address:</strong>
                    <div t-field="o.partner_id" 
                        t-field-options='{"widget": "contact", "fields": ["address", "name", "phone", "fax","email","vat"], "no_marker": false}'/>
                    <p t-if="o.partner_id.vat">VAT: <span t-field="o.partner_id.vat"/></p>
            </td>
        </tr>
        </table>

            <div class="row mt32 mb32" id="informations">
                <div t-if="o.client_order_ref" class="col-xs-3">
                    <strong>Invoice:</strong>
                    <p t-field="o.client_order_ref"/>
                </div>
                <div t-if="o.user_id.name" class="col-xs-3">
                    <strong>Salesperson:</strong>
                    <p t-field="o.user_id.name"/>
                </div>
                <div t-if="o.payment_term" class="col-xs-3">
                    <strong>Payment Term:</strong>
                    <p t-field="o.payment_term"/>
                </div>
            </div>

</template>

```

### 3. 创建自定义的渲染函数
有两个方法
### 方法 1 
odoo 使用这个方法重用7版代码

```
import time
from openerp.report import report_sxw
from openerp.osv import osv

class sale_report_xxx(report_sxw.rml_parse):
    def _print_test(self):
        return "good"

    def __init__(self, cr, uid, name, context):
        super(sale_report_libiya, self).__init__(cr, uid, name, context=context)
        self.localcontext.update({
            'time': time,
            'cr':cr,
            'uid': uid,
            'curr_rec': self.curr_rec,
            'compute_currency': self.compute_currency,
            'print_test': self._print_test,
            'print_test2': "good2",
            'other_methods'self._other_methods,
        })

class report_pos_details(osv.AbstractModel):
    _name = 'report.sale_webkit_report_libiya.report_sale_order_xxx'
    _inherit = 'report.abstract_report'
    _template = 'module.report_sale_order_xxx'
    _wrapped_report_class = sale_report_xxx
```

#### 方法 2
（  odoo 官方文档的代码例子）

```
from openerp import api, models


class ParticularReport(models.AbstractModel):
    _name = 'report.<<module.reportname>>'
    @api.multi
    def render_html(self, data=None):
        report_obj = self.env['report']
        report = report_obj._get_report_from_name('<<module.reportname>>')
        docargs = {
            'doc_ids': self._ids,
            'doc_model': report.model,
            'docs': self,
        }
        return report_obj.render('<<module.reportname>>', docargs)
```

## 小工具
### 报表网页编辑工具
安装 website_editor模块 , 在后台修改报表类型为HTML后，website manager 用户可以在线修改报表样式。

网页编辑完后，报表类型调整回pdf，即可再次答应pdf。

## 索引
* [odoo文档](https://www.odoo.com/documentation/8.0/reference/qweb.html)
* [bootstrap grid 语法](http://www.w3cschool.cc/bootstrap/bootstrap-grid-system.html)

