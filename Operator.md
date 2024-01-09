# Operator

## AND OR NOT Operator

AND 总是被优先评估。执行查询时候，查询执行引擎首先评估这个条件（第四行）。

```sql
SELECT *
FROM customers
where birth_date > '1990-01-01' OR 
points > 1000 AND state = 'VA'
```

## IN Operator

```sql
SELECT *
FROM customers
WHERE state IN ('VA','FL','GA')

SELECT *
FROM customers
WHERE state NOT IN ('VA','FL','GA')
```

## BETWEEN Operator

```sql
SELECT *
FROM customers
WHERE birth_date between '1990-1-1' and '2000-1-1';
```

## LIKE Operator

```sql
#找到所有b开头的。（大小写不敏感）。
SELECT *
FROM customers
WHERE last_name LIKE 'b%';

#找到所有有b的。（大小写不敏感）。
SELECT *
FROM customers
WHERE last_name LIKE '%b%';

#找到所有两个字符，第一个随意，第二个为y的。（大小写不敏感）。
SELECT *
FROM customers
WHERE last_name LIKE '_y';

-- % any number of characters
-- _ single character
```

## REGEXP Operator

正则表达式。

```sql
-- 找 包含field的字符串
SELECT *
FROM customers
WHERE last_name regexp 'field'

-- 可以用^表示字符串的开头
SELECT *
FROM customers
WHERE last_name regexp '^field'

-- 可以用$表示字符串的结尾
SELECT *
FROM customers
WHERE last_name regexp 'field$'

-- 可以用|表示别的模式
SELECT *
FROM customers
WHERE last_name regexp 'field|mac'


-- 可以用[]表示存在区间里的元素
SELECT *
FROM customers
WHERE last_name regexp '[gim]e'
-- ge me ie
WHERE last_name regexp '[a-h]e'
-- ae be ce ... he

-- 可以用|表示别的模式
SELECT *
FROM customers
WHERE last_name regexp 'field|mac'
```

## is NULL Operator

```
SELECT *
FROM customers
WHERE phone is NULL;
```

# 子句

## Order By

```sql
SELECT *
FROM customers
order by first_name; -- 升序

SELECT *
FROM customers
order by first_name DESC; -- 升序

SELECT *
FROM customers
order by state DESC,first_name; -- 混合型。
```

有意思的联系

```sql
SELECT *,quantity*unit_price as total_price
FROM sql_store.order_items
WHERE order_id = 2
ORDER BY total_price DESC;
```

## LIMIT 子句

```SQL
SELECT *
FROM customers
LIMIT 300
-- 拿前300行，分页时有用。

SELECT *
FROM customers
LIMIT 6，3
-- 跳过前6条然后拿3条
```

# 连接

## Inner Joins

```sql
SELECT order_id,o.customer_id,first_name,last_name
FROM orders o
JOIN customers c
		ON o.customer_id = c.customer_id
-- 多个表出现同样的列名，需要在前加上表名以确定。
-- 可以在表后加上别名。
```

## Joining Across DataBase

```sql
SELECT *
FROM order_items oi
JOIN sql_inventory.products p
		ON oi.product_id = p.product_id
-- 只需要给不再当前数据库里的表加上数据库前缀。
```

## Self Joins

```sql
SELECT 
		e.employee_id,
        e.first_name,
        m.first_name AS manager
FROM employees e
JOIN employees m
		ON e.reports_to = m.employee_id
-- 找每个员工的老板。
```

## Joining Multiple Tables

```sql
USE sql_store;

SELECT 
		o.order_id,
        o.order_date,
        c.first_name,
        c.last_name,
        os.name AS status
FROM orders o
JOIN customers c
		ON o.customer_id = c.customer_id
JOIN order_statuses os
		ON o.status = os.order_status_id
		
-- 联合了s
```

## Compound Join Condition

复合主键。

```sql
SELECT *
FROM order_items oi
JOIN order_item_notes oin
		ON oi.order_id = oin.order_id
        AND oi.product_id = oin.product_id;
```

## Implicit Join Syntax

尽量用显式连接。

## Outer Joins

也就是左连接和右连接。

LEFT JOIN AND RIGHT JOIN

```sql
-- 左连接，无论后面返回对还是错，都将左表返回。
SELECT 
		c.customer_id,
        c.first_name,
        o.order_id
FROM customers c
LEFT JOIN orders o
		ON c.customer_id = o.customer_id
ORDER BY c.customer_id
```



```sql
-- 右连接，无论后面返回对还是错，都将右表返回。
SELECT 
		c.customer_id,
        c.first_name,
        o.order_id
FROM customers c
RIGHT JOIN orders o
		ON c.customer_id = o.customer_id
ORDER BY c.customer_id
        
```

## Outer Join Between Multiple Tables

```sql
SELECT 
		c.customer_id,
        c.first_name,
        o.order_id,
        sh.name AS shipper
FROM customers c
LEFT JOIN orders o
		ON c.customer_id = o.customer_id
LEFT JOIN shippers sh
		ON o.shipper_id = sh.shipper_id
ORDER BY c.customer_id
```

## Self Outer Joins

```sql
-- 找CEO

SELECT 
	e.first_name
FROM employees e
LEFT JOIN employees m
		ON e.reports_to = m.employee_id
WHERE e.reports_to is NULL
```

## The USING Clause

```sql
-- 使用相同列名，可以用USING子句。
SELECT 
	o.order_id,
    c.first_name,
    sh.name
FROM orders o
JOIN customers c
	-- ON o.customer_id = c.customer_id
		USING (customer_id)
LEFT JOIN shippers sh
		USING (shipper_id)
```

## Natural Joins

```sql
-- 根据相同的列合并
SELECT 
		o.order_id,
		c.first_name
FROM orders o
NATURAL JOIN customers c
```

mosh不建议。

## Cross Join

求笛卡尔积，左侧表每一行都会和右侧表每一行连接建立一行。

颜色表，求规格。大中小。

```sql
-- 这个是显式连接
SELECT 
		c.first_name AS customer,
        p.name as name
FROM customers c
CROSS JOIN products p

-- 这个是隐式连接
SELECT 
		c.first_name AS customer,
        p.name as name
FROM customers c，products p
```

## Unions

使用Union，可以合并多段查询的记录。

```sql
SELECT 
		customer_id,
        first_name,
        points,
        'Bronze' as type
FROM customers
WHERE points <= 2000
UNION
SELECT 
		customer_id,
        first_name,
        points,
        'Silver' as type
FROM customers
WHERE points BETWEEN 2000 and 3000
UNION
SELECT 
		customer_id,
        first_name,
        points,
        'Gold' as type
FROM customers
WHERE points >= 2000
ORDER BY first_name
```

# 列属性

## Column Attributes

## Insert A Row

```sql
INSERT INTO customers
VALUES (DEFAULT,
        'John',
        'Smith',
        '1990-01-01',
        NULL,
        'address',
        'city',
        'CA',
        200)
-- 加了一行，也可以在customer后加括号。
INSERT INTO customers(
		last_name,
        first_name,
        birth_date,
        address,
        city,
        state)
VALUES (
    'John',
    'Smith',
    '1990-01-01',
    'address',
    'city',
    'CA')
```

## Insert Muliple Rows

```sql
INSERT INTO products (name,quantity_in_stock,unit_price)
VALUES ('Product1',10,1.95 ),
				 ('Product2',11,1.95),
                 ('Products3',12,1.95) 
```

## Inserting Hierarchical Rows

订单表的一行可以在订单项目表中有1子或者多子。

```SQL
INSERT  INTO orders (customer_id,order_date,status)
VALUES (1,'2019-01-02',1);
INSERT INTO order_items
VALUES (last_insert_id(),1,1,2.95),
				(LAST_INSERT_ID(),2,1,3.95)
				
-- 函数last_insert_id() 能返回上一次插入的主键id
```

## Creating a Copy of a Table

```sql
-- 快速建表的技巧，但是这样做，新复制的表格不具备之前表格的一些属性。会有问题。
CREATE TABLE XX AS SELECT子句


CREATE TABLE invoices_archieve AS
(SELECT p.invoice_id,
				p.number,
                c.name,
                p.invoice_total,
                p.payment_total,
                p.invoice_date,
                p.due_date,
                p.payment_date
FROM (SELECT *
				FROM invoices
				WHERE payment_date is NOT NULL) p
JOIN clients c 
		ON p.client_id = c.client_id)
```

## Updating a Single Row

```sql
UPDATE invoices
SET payment_total = 10,payment_date = '2019-03-01'
WHERE invoice_id = 1
```

## Updating Multiple Rows

```sql
UPDATE customers
SET points = points+50
WHERE birth_date < '1990-01-01'
```

## Using Subqueries in Updates

```sql
UPDATE orders 
SET comments = 'Gold customer'
WHERE customer_id IN (SELECT customer_id FROM customers WHERE points > 3000)
```

## Deleting Rows

```sql
DELETE FROM invoices
WHERE client_id = (SELECT client_id FROM clients WHERE name = 'Myworks')
```

## Restoring the Databases

就是重新运行一遍脚本。

# 聚合函数

## Aggregate Function 

只运行非空值，空值不会被计算。

用*号可以计算总共记录数。会取重复值，如果不想要重复值，必须DISTINCT。

```sql
SELECT 
		MAX(invoice_total) AS highest,
        MIN(invoice_total) AS lowest,
        AVG(invoice_total) AS average,
        SUM(invoice_total * 1.1) AS total,
        COUNT(invoice_total ) AS number_of_invoices,
        COUNT(payment_date) AS number_of_paymented,
        COUNT(DISTINCT client_id) AS total_client,
        COUNT(*) AS total_records
FROM invoices
WHERE invoice_date > '2019-07-01'

```

## Group BY Clause

语句执行顺序：**FROM -> WHERE ->GROUP BY - > SELECT -> ORDER BY**

```sql
SELECT 
		state,
        city,
        SUM(invoice_total) AS total_sale
FROM invoices i
JOIN clients USING (client_id)
GROUP BY state,city
-- GROUP 有多个对象是，每个state 和 city组合 得到一台记录。
```

```sql
SELECT 
		p.date,
        pm.name AS payment_method,
        SUM(p.amount) AS total_payment
FROM payments p
JOIN payment_methods pm
		ON p.payment_method = pm.payment_method_id
GROUP BY p.date,pm.name
ORDER BY p.date
```

## The Having Clause

```sql
SELECT c.last_name,SUM(quantity*unit_price) AS total_payment
FROM customers c
JOIN orders USING (customer_id) 
JOIN order_items USING (order_id)
WHERE state = 'VA'
GROUP BY customer_id
HAVING SUM(quantity*unit_price) >100
```

这个没做好。

## The ROLLUP Operator

起到一个分类汇总的作用

```SQL
-- 最后加一行汇总。
SELECT 
		client_id,
        SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id WITH ROLLUP

-- 分类汇总，对于每个state也有一个关于所有城市的汇总。
SELECT
		state,
        city,
        SUM(invoice_total) AS total_sales
FROM invoices i
JOIN clients c USING (client_id)
GROUP BY state,city WITH ROLLUP

-- emm 使用rollup 前面的GROUP BY 不能使用列别名payment_method
SELECT 
		pm.name AS payment_method,
        SUM(p.amount) as total
FROM payments p
JOIN payment_methods  pm ON p.payment_method = pm.payment_method_id
GROUP BY pm.name with rollup
```

# 编写复杂查询

## Subqueries

```sql
SELECT e.first_name,e.last_name,salary
FROM employees e
WHERE salary > (SELECT AVG(salary)
				FROM employees e)
UNION
SELECT '','',AVG(salary)
FROM employees e
```

## The IN Operator

```sql
SELECT product_id,name
FROM products p
WHERE p.product_id NOT IN (SELECT DISTINCT product_id
														FROM order_items)
```

## Subqueries VS Joins

```SQL
-- 子查询
SELECT 
		c.customer_id,
        c.first_name,
        c.last_name
FROM customers c
WHERE c.customer_id IN (SELECT DISTINCT customer_id
                        FROM orders o
                        WHERE order_id in (SELECT order_id
                       						FROM order_items
                                           WHERE product_id = 3))
-- join   
SELECT 
		distinct customer_id,
        c.first_name,
        c.last_name
FROM customers c
JOIN orders  o USING (customer_id)
JOIN order_items oi USING (order_id)
WHERE product_id = 3 
```

## The ALL Keyword

```SQL
SELECT *
FROM invoices
WHERE invoice_total > ALL(SELECT invoice_total
						FROM invoices
						WHERE client_id = 3)
-- ALL 比较里面所有的。属于是一种抽象思维。而max是一种具象思维。
```

## The Any Keyword

```SQL
-- 跟any类似等价。
SELECT *
FROM clients
WHERE client_id = ANY ( 
		SELECT client_id
        FROM invoices
        GROUP BY client_id
        HAVING COUNT(*) >=2 )
```

## Correlated Subqueries

```sql
-- 同一行表，分两次看。
SELECT *
FROM employees e
WHERE salary > (
		SELECT AVG(salary)
        FROM employees
        WHERE office_id = e.office_id)
```

## The EXISTS Operator

EXIST能有效提高效率。

```sql
-- 返回True就会有这条，false就没有。
SELECT *
FROM clients c
WHERE EXISTS (
		SELECT client_id
        FROM invoices
        WHERE client_id = c.client_id)
```

```sql
-- 找没被订过的商品
SELECT *
FROM products p
WHERE NOT EXISTS (SELECT *
                    FROM order_items oi
                    WHERE p.product_id = oi.product_id )
```

## Subqueries in the SELECT Clause

```sql
-- 
SELECT invoice_id,invoice_total,
		(SELECT AVG(invoice_total) FROM invoices ) AS invoice_average,
        invoice_total - (SELECT invoice_average) AS difference
FROM invoices 

SELECT client_id,name,
					(SELECT SUM(invoice_total) 
								FROM invoices WHERE  client_id = c.client_id) AS total_sales,
					(SELECT AVG(invoice_total)
								FROM invoices) AS average,
					(SELECT total_sales- average) AS difference
FROM clients c
```

## Subqueries in the FROM Clause

from子句的查询必须要有一个别名，这是必须的。

```SQL
SELECT *
FROM (
SELECT client_id,name,
					(SELECT SUM(invoice_total) 
								FROM invoices WHERE  client_id = c.client_id) AS total_sales,
					(SELECT AVG(invoice_total)
								FROM invoices) AS average,
					(SELECT total_sales - average) AS difference
FROM clients c
) AS sales_summary
WHERE total_sales is NOT NULL
```





















