## 1.Numeric Functions

```SQL
-- 四舍五入
SELECT ROUND(5.49) #5
SELECT ROUND(5.5) #6
-- 保留一位小数
SELECT ROUND(5.49,1)#5.5

-- 返回大于等于这个数的最小整数
SELECT CEILING(5.7) #6

-- 返回小于等于这个数的最大整数
SELECT FLOOR(5.7) #5

-- ABS绝对值
SELECT FLOOR(-5.7) #5.7

-- 0.1随机浮点数
SELECT RAND()
```

## 2.String Functions

```sql
-- 获得全大写
SELECT UPPER('sky')

-- 获得全小写
SELECT LOWER('Sky')

-- LTRIM 删除左侧空格
SELECT LTRIM('   SKY')

-- RTRIM 删除右侧空格
SELECT RTRIM('SKY   ')

-- TRIM 删除左右侧空格
SELECT TRIM('    SKY   ')

-- 返回字符串左侧4个字符
SELECT LEFT('Kindergarden',4)

-- 返回字符串右侧4个字符
SELECT RIGHT('Kindergarden',4)

-- 第二个参数是位置，第三个参数是长度
SELECT SUBSTRING('Kindergarden',3,5) #第一个位置是1

-- 第二个参数是位置，从第3个字符开始到最后
SELECT SUBSTRING('Kindergarden',3) #第一个位置是1

-- 从左到右找字符，大小写不区分，找不到返回0
SELECT LOCATE('n','KiNdergarden')

-- 从左到右找字符串，大小写不区分，找不到返回0
SELECT LOCATE('garden','KiNdergarden')

-- 替换 第二个参数是找，第三个是替换的。
SELECT REPLACE('KiNdergarden','garden','GARTEN') 

-- 连接
SELECT CONCAT('first','last')
SELECT CONCAT(first_name,' ',last_name) AS full_name
FROM customers
```

## 3.Date Functions

```SQL
-- 显示当前日期加时间
SELECT NOW()
-- 显示当前日期
SELECT CURDATE()
-- 显示当前时间
SELECT CURTIME()

-- YEAR函数可以获得当前日期的年份，哪怕是类日期也能获取
-- DAY函数可以获得当前日期的号，比如11月14号，返回14
-- HOUR函数可以获得小时，比如11：02，返回11
-- MINUTE函数可以获得小时，比如11：02，返回11
-- SECOND函数可以获得小时，比如11：02，返回11
SELECT YEAR(NOW())
SELECT DAY(NOW())
SELECT HOUR(NOW())

-- 返回星期几，比如星期一会返回Monday
SELECT DAYNAME(NOW())
-- 三月返回March
SELECT MONTHNAME(NOW()) 

-- 提取
SELECT EXTRACT(YEAR FROM NOW()) -- 提取年
SELECT EXTRACT(MONTH FROM NOW()) -- 提取月份
SELECT EXTRACT(DAY FROM NOW()) -- 提取日期
```

## 4.Formatting Dates and Times

```SQL
-- 用两位表示年份，%y。用四位表示年份，%Y
SELECT DATE_FORMAT(NOW( ),'%y') 

-- 用两位表示月份，%m。用名称表示月份，%M
SELECT DATE_FORMAT(NOW( ),'%m') 

-- 用两位表示日，%d。
SELECT DATE_FORMAT(NOW( ),'%d') 

SELECT TIME_FORMAT(NOW(),'%H:%i %p')
-- 输出 17:25 PM
```

## 5.Calculating Dates and Times

应用场景：加一小时，加一天，计算某两时间之间的间隔。

```sql
-- 明天的同一时间 , 同理：DAY,MONTH,YEAR,MINUTE,SECOND...
-- 这是增加，其实如果换成负数就是往回退。
-- 另一个回退就是使用函数DATE_SUB(),传递正值。
SELECT DATE_ADD(NOW(),INTERVAL 1 DAY) 

-- 计算时间间隔 输出4 指间隔四天
SELECT DATEDIFF('2019-01-05 09:00','2019-01-01 17:00')
TIME_TO_SEC()函数，返回从零点流经的秒数。由此可以计算。
```

## 6.IFNULL and COALESCE Functions

```sql
-- 如果shipper_id为NULL，返回第二个。
SELECT 
		order_id,
        IFNULL(shipper_id,'Not assigned') AS shipper
FROM orders

-- COALESCE 第一个不为NULL，返回第二个，第二个不为NULL，返回第三个。。这样持续下去。（返回这里第一个不为空的值）
SELECT 
		order_id,
        COALESCE(shipper_id,comments,'Not assigned') AS shipper
FROM orders
```

## 7.The IF Function

```sql
-- if(t,a,b) ,return t=true?a:b;
SELECT 
	product_id,
    name,
    COUNT(*) AS orders,
    if(COUNT(*) > 1,'Many times','Once') AS Frequency
FROM products
JOIN order_items USING (product_id)
GROUP BY product_id
```

## 8.The Case Operator

```sql
SELECT 
		order_id,
        CASE 
				WHEN YEAR(order_date) = YEAR(NOW()) THEN 'Active'
                WHEN YEAR(order_date) = YEAR(NOW()) - 1 THEN 'Last Year'
                WHEN YEAR(order_date) <YEAR(NOW()) - 1 THEN 'Archived'
                ELSE 'Future'
		END AS category
FROM orders

SELECT 
		CONCAT(first_name,' ',last_name),
        points,
        CASE 
				WHEN points > 3000 THEN 'Gold'
                WHEN points between 2000 and 3000 THEN  'Silver'
                WHEN points < 2000 THEN 'Bronze'
                ELSE 'Future'
		END AS category
FROM customers
ORDER BY points DESC
```

