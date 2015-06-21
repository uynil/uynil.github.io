---
layout: post
title: Qweb report introduction
tags: [odoo]
---

#{{page.title}}

### {{ page.date | date_to_string }}
Before start of introduction, odoo (former openerp) is open source, the source code is the best tutorial.

Eg, you can check point_of_sale module to check out how to write qweb report.

# qweb report introduction
openerp v7 use webkit and rml, openerp v6 uses rml.
qweb report is the new report engine of odoo V8, webkit and rml is depreciated from v8.

qweb is also rendering engine for odoo web server. Thus, odoo standardise the web engine.


## Guide line

This article talks about

* qweb report introduction
* how to create a qweb report

## qweb report
### main elements
	** report registration in database
	** report translation registration in database (optional)
	** report layout in database
	** customize rendering functions (optional)	

### Qweb introduction
In general , 

1. developer write xml code to create actions, report record, views, web layout.
2. when installing modules, there actions, reports, views is saved in database.
3. user click a button, open a new windows, it trigger the actions and Web javascript API. Web javascript will 
    * find the view id and read layout code from databass render it into web
    * find record id and read record value from database render it into web

4. when print report , odoo will trigger wkhtmltopdf library to render web into pdf document.

### odoo qweb introduction
#### grammer
* data
	`t-field`, `t-esc`
 
* loop, condition

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

## how to create a qweb report
### 0. moduel structure

```
module
	| report
		|	customize_report.py
	| views
		|	report_layout_view.xml
	| report.xml
	| __init__.py
	| __openerp__.py
	| ...
```
### 1. create a report

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
### 2. create a report translation (optional)
```
<template id="report_sale_order_xxx">
    <t t-call="report.html_container">
        <t t-foreach="doc_ids" t-as="doc_id">
            <t t-raw="translate_doc(doc_id, doc_model, 'partner_id.lang', 'module.report_sale_order_xxx_document')"/>
        </t>
    </t>
</template>
```

### create report layout

odoo is using bootstrap for web layout

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

### create customize report rending function
There are two ways to registrating 

### method 1 
odoo use this way to reuse the code of v7.

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

#### Method 2
code taken from odoo documentation

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

## tips
### website editor
After this module installed, website manager can edit report online once report type is set in backend  to html.

After the online edit, the report can be changed back to pdf to print pdf document later.

## Refs
* [odoo documentation](https://www.odoo.com/documentation/8.0/reference/qweb.html)
* [bootstrap grid](http://www.w3cschool.cc/bootstrap/bootstrap-grid-system.html)

## [Elico Corp (shenzhen) is hiring][job_link]
[job_link]: http://simple-is-better.com/jobs/866 "Eilco Shenzhen hire odoo developers"
