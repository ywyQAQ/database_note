## Creating Views

```sql
CREATE VIEW sales_by_client AS (
SELECT c.client_id,
				c.name,
                SUM(invoice_total) AS total_sales
FROM clients c
JOIN invoices i USING (client_id)
GROUP BY client_id,name)

-- 这个view可以当表来使用。
-- 但view不存储数据
```

## Altering or Dropping Views

```sql
DROP VIEW sales_by_client

-- 如果有了就重置，没有就创造。
CREATE OR REPLACE VIEW sales_by_client AS ...

```

## Updatable Views

Views也可以被更新，但是仅局限特定情况（没有DISTINCT,Aggregate Functions，GROUP BY /HAVING/UNION），即可以在INSERT UPDATE DELETE语句中使用视图。

另外，当为view插入表时，view不为空的列一定要有，（比如日期）因为如果表要求日期不为空，插入视图没有日期，那么数据就会出错，因为不允许。

同时删除的时候，会将视图来自的表table中的列也删掉。个人感觉很危险。但实际上开发时候，自己往往没有直接接触表的权限，只能通过视图来处理。

## The WITH OPTION CHECK Clause

使用WITH CHECK OPTION 可以防止行突然消失。

比如这个视图是要求x > 1 ，我将一行的 x 置为 0 ，如果不写WITH CHECK OPTION 这行就会消失，如果写了，在置零的时候会报错。

```sql
CREATE OR REPLACE VIEW invoices_with_balance AS
SELECT 
		invoice_id,
        number,
        client_id,
        invoice_total,
        payment_total,
        invoice_total - payment_total AS balance,
        invoice_date,
        due_date,
        payment_date
FROM invoices
WHERE (invoice_total - payment_total) > 0
WITH CHECK OPTION
```

## Other Benefits of Views

### 1.Simpligy queries

### 2.Reduce the impact of changes

意思就是大家都用视图作为查询的依据，我如果想改一列的名字，不需要改所有的查询，只要改视图，列的别名用之前的原名字，就ok了。

### 3.Restrict access to the data

不让随便改原数据。