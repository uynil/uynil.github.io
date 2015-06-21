---
layout: post
title: odoo tdd 初探
---

TDD 测试驱动

最近我在用TDD开发一个框架，深切的体会就是，除了写测试、写代码、重构三种状态。
TDD的具体应用实例， 就是XP极限编程，TDD + 重构。

odoo tdd 的意义讨论。
之所以为了tdd，一是现在我司项目定制越来越多，开发需要质量管控，质量管控在开发中就体现在unit test，unit test的最好实践模式就为 tdd，二是 tdd，是当今敏捷，寰享科技深圳公司希望实践敏捷，实现持续交付，tdd就是其中开发环节必不可少的一环。
本文最后我们会就odoo的 tdd 的可行性和必要性做一个总结。

# 更新
* 150613
    单元测试粒度不用太细，原则以建立不同场景的测试建立不同测试, 以程序单元外部接口为主,尽量不要为细分函数建立测试。
    举例:  excel导入单元测试中，程序上只需要测试excel导入后，生成picking的函数即可,如果必须细分程序到excel导入，再细分为excel导入、数据准备等函数;情境上，导入excel数据的不同情形可以多见几个测试。

# 测试
软件开发其他阶段的测试驱动开发，根据测试驱动开发的思想完成对应的测试文档即可。下面针对详细设计和编码阶段进行介绍。
测试驱动开发的基本过程如下：
1） 明确当前要完成的功能。可以记录成一个 TODO 列表。
2） 快速完成针对此功能的测试用例编写。
3） 测试代码编译不通过。
4） 编写对应的功能代码。
5） 测试通过。
6） 对代码进行重构，并保证测试通过。
7） 循环完成所有功能的开发。
为了保证整个测试过程比较快捷、方便，通常可以使用测试框架组织所有的测试用例。一个免费的、优秀的测试框架是 Xunit 系列，几乎所有的语言都有对应的测试框架。我曾经写过一篇文章介绍CppUnit的文章（ http://www.ibm.com/developerworks/cn/linux/l-cppunit/index.html）。
开发过程中，通常把测试代码和功能代码分开存放，这里提供一个简单的测试框架使用例子，您可以通过它了解测试框架的使用。下面是文件列表。


# tdd 实例

http://www.iteye.com/topic/6551

# odoo TDD 实例

用户需要在现有ERP系统中开发一个群发报价邮件给所有客户或询盘邮件给供应商。
经过讨论，考虑到一次要群发的邮件已以千记，如果邮件即时发送，会造成发送较慢。
实现的方案，利用odoo 邮件定时发送功能，发送邮件时只发送邮件到

### 0. odoo unit test 准备工作

linux 开发服务器

https://www.odoo.com/documentation/8.0/reference/testing.html
https://www.odoo.com/forum/how-to/developers-13/how-to-run-tests-526

>  Tests are automatically run when installing or updating modules if --test-enable was enabled when starting the Odoo server.
> As of Odoo 8, running tests outside of the install/update cycle is not supported.



以更新模块 (-d dbname -u module_name)和测试模式 (--test-enable --log-level=test) 启动odoo 服务器。

./openerp-server -c /path/to/file.conf -d dbname -u module_name --test-enable --log-level=test


### 1. 细化功能

1. create menu, action, wizard

创建按钮，动作，视图。没有走python unit。
也没有走 web test。

不过感觉 web test 还用来验证。

另外，交互设计，入口设计 可以走 高保真纸上原型 来验证。

2. create wizard view


3. create email from wizard
3.1 创建 第一个测试函数和原型函数。

test_create_email

```
class TestMassEmail(TransactionCase):
    def test_create_email(self):
        val = {
            'customer': True,
            'supplier': False,
        }
        wizard_ids = self.env['mass.email.supplier_wizard'].send_email(val)
        self.assertEqual(
            1,
            len(wizard_ids))
```

wizard
```
class mass_email_suppliercustomer(osv.osv_memory):
    '''
    Mass Email to  Supplier Customer
    '''
    _name = 'mass.email.suppliercustomer'
    _descript = 'Mass Email to Supplier Customer'
    _columns = {
        'supplier': fields.boolean("supplier"),
        'customer': fields.boolean("customer"),
    }

    def send_email(self, cr, uid, ids, context=None):
        mail_pool = self.pool.get('mail.mail')
        partner_pool = self.pool.get('res.partner')
        res = []
        _logger.info("\n\n\n Send Email \n\n")
        return res
```

3.2 添加 wizard 属性

test

```
class TestMassEmail(TransactionCase):
    def test_create_email(self):
        cr, uid = self.cr, self.uid
        val = {
            'customer': True,
            'supplier': False,
            'subject': "Mass test",
            'body': "<p>Mass Test Body</p>",
            "attachment_ids": [1],
        }
        wizard_ids = self.env['mass.email.suppliercustomer'].create(val)
        wizard_ids = [wizard_ids[0]]
        self.assertEqual(
            1,
            len(wizard_ids))
        _logger.info("\n\n\n Test Create Email \n\n")

        mail_ids = self.registry('mass.email.suppliercustomer').send_email(cr, uid, wizard_ids)
        self.assertEqual(
            47,
            len(mail_ids))
```

wizard
```
class mass_email_suppliercustomer(osv.osv_memory):
    '''
    Mass Email to  Supplier Customer
    '''
    _name = 'mass.email.suppliercustomer'
    _descript = 'Mass Email to Supplier Customer'
    _columns = {
        'supplier': fields.boolean("supplier"),
        'customer': fields.boolean("customer"),
        'subject': fields.char("Subject"),
        'body': fields.html("Body"),
        'attachment': fields.html("Attachment"),
        'attachment_ids': fields.many2many('ir.attachment',
            'mail_compose_message_ir_attachments_rel',
            'wizard_id', 'attachment_id', 'Attachments'),
    }


    def send_email(self, cr, uid, ids, context=None):
        mail_pool = self.pool.get('mail.mail')
        partner_pool = self.pool.get('res.partner')
        res = []
        for wizard in ids:
            val = {
                'subject': wizard.subject,
                'body_html': wizard.body,
                'attachment_ids': wizard.attachment_ids
            }
        #     if wizard.customer:
        #         customer_ids = partner_pool.search(
        #             cr, uid, [('customer', '=', True)])
        #         val.update({'recipient_ids': customer_ids})
        #         res = mail_pool.create(cr, uid, val, context)
        #     if wizard.supplier:
        #         supplier_ids = partner_pool.search(
        #             cr, uid, [('supplier', '=', True)])
        #         val.update({'recipient_ids': supplier_ids})
        #         res_ids = mail_pool.create(cr, uid, val, context)
        return res
```

3.3 创建 客户邮件测试

测试完删除测试数据

```
    def test_create_client(self):
        cr, uid = self.cr, self.uid
        val = {
            'customer': True,
            'supplier': False,
            'subject': "Mass test",
            'body': "<p>Mass Test Body</p>",
            "attachment_ids": [1],
            'state': 'outgoing',
        }
        wizard_ids = self.env['mass.email.suppliercustomer'].create(val)
        wizard_ids = [wizard_ids.id]
        self.assertEqual(
            1,
            len(wizard_ids))
        _logger.info("\n\n\n Test Create Client \n\n")

        mail_ids = self.registry('mass.email.suppliercustomer').send_email(cr, uid, wizard_ids)
        self.assertEqual(
            47,
            len(mail_ids))
        self.env['mail.mail'].unlink()
```

3.4 创建 客户邮件测试

test case

```
    def test_create_client(self):
        cr, uid = self.cr, self.uid
        val = {
            'customer': True,
            'supplier': False,
            'subject': "Mass test",
            'body': "<p>Mass Test Body</p>",
            "attachment_ids": [1],
        }
        wizard_ids = self.env['mass.email.suppliercustomer'].create(val)
        wizard_ids = [wizard_ids.id]
        self.assertEqual(
            1,
            len(wizard_ids))
        _logger.info("\n\n\n Test Create Customer \n\n")

        mail_ids = self.registry('mass.email.suppliercustomer').send_email(cr, uid, wizard_ids)
        self.assertEqual(
            47,
            len(mail_ids))
        self.env['mail.mail'].unlink()

    def test_create_supplier(self):
        cr, uid = self.cr, self.uid
        val = {
            'customer': False,
            'supplier': True,
            'subject': "Mass test Supplier",
            'body': "<p>Mass Test Body</p><br/> Supplier",
            "attachment_ids": [1],
        }
        wizard_ids = self.env['mass.email.suppliercustomer'].create(val)
        wizard_ids = [wizard_ids.id]
        self.assertEqual(
            1,
            len(wizard_ids))
        _logger.info("\n\n\n Test Create Supplier \n\n")

        mail_ids = self.registry('mass.email.suppliercustomer').send_email(cr, uid, wizard_ids)
        self.assertEqual(
            24,
            len(mail_ids))
        self.env['mail.mail'].unlink()

    def test_create_customer_supplier(self):
        cr, uid = self.cr, self.uid
        val = {
            'customer': True,
            'supplier': True,
            'subject': "Mass test Supplier",
            'body': "<p>Mass Test Body</p><br/> Customer Supplier",
            "attachment_ids": [1],
        }
        wizard_ids = self.env['mass.email.suppliercustomer'].create(val)
        wizard_ids = [wizard_ids.id]
        self.assertEqual(
            1,
            len(wizard_ids))
        _logger.info("\n\n\n Test Create Customer Supplier \n\n")

        mail_ids = self.registry('mass.email.suppliercustomer').send_email(cr, uid, wizard_ids)
        self.assertEqual(
            71,
            len(mail_ids))
        self.env['mail.mail'].unlink()
```

4. test sending mail scheduler 


### 

### -1 回顾 总结


# odoo tdd 小结

* 通过 tdd 实现开发思路的整理。
tdd是一个编程思路，没有tdd的习惯，直接写功能函数。

* odoo前台编程，是简单的xml编写，后台渲染，所以可以快速验证，单元测试没有必要。
v8的qweb 报表，v7的rml报表也是一样。

* 通常情况下，编程时，我会加上一些print 语句，判断关键节点的变量。

* tdd 下，前台 后台的顺序可以点到，先编写后台，再写前台。


## [寰享科技(深圳) 正在招聘 初级/高级odoo 技术工程师][job_link]
[job_link]: http://simple-is-better.com/jobs/866 "Eilco Shenzhen hire odoo developers"

如果有任何疑问，欢迎发邮件到 uynill#163.com 跟我讨论。（# 改成@）
