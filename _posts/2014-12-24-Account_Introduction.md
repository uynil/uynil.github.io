# odoo accounting

## Introduction

### Real world accounting
In real accounting system, based on document（ bank statements, other original invoice, tax reports, etc），accountant will create account move manually.

### odoo accounting
In odoo，user input documents ( invoice，payment, expenses, assets...), system will generate account vouchers accordingly.

0. Financial manager configure system, so that system can generate account moves automatically.
1. cashier / assistant create invoice, bank payment,
2. accountant check and post accounting entries.
3. Generate Accounting Reports ( P&L)

In the following, we will check thse points

## Accounting set up

* currency

	desactive currencies expect those will be used.
* accounts

	We use standard accounts code for China,
	We use HK accounts for HongKong.

	Create child account for AR, AP, salary tax, bank, expense accounts, in chinese practise, we add two postfix number for account code.
	
	Eg, a child account "管理费用-Rent" code 660202 under 6602 "管理费用"

* Products

	odoo use configuration of product to generate automatic entrires.
	user should create one product for each expense type, and set up proper accounts for this product.

```
	Eg under chinese accounting system, if we want to generate expenses for "expense rent"
	we create a child account "管理费用-rent" code 660202 under 6602 "管理费用".
	we create a new product "Rent" with expense account 660202 "管理费用-rent"
```
	odoo will create account moves based on
		the set up and use of products
		choose of accounts after in the invoice / expense
	
* assets

	Configure assets type in menu "configuration / asset types".

## Input documents
User input their expense, invoice, payments, assets
	* with proper products to generate correct moves in correct accounts
	* change proper accounts if needed

### Other account moves 
For other accounting moves,
	please create them manually if needed.

## Post entries
Users go to menu “Accounting / Account Moves”to pose entires. 

## Reports

### P & L
One column P&L report.

### Balance Sheet
Two Column Accounts Balance sheet.

## Conclusion

With odoo, accountant will be in charge of

* verify accounting documents
* post accounting entries
* input manual accounting entries if needed
* generate accounting reports
