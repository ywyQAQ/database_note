## Data Modelling

### Understand the requirements

### Build a Conceptual Model

Represents the entities and their relationships

UML 或 ER 图

### Build a Logical Model

指明数据类型，比如string int

关系： 1v1 1vp pvp （其实没有多对多，如果有多对多，那么需要重新一个连接表，实现两个1对多）

同时写出了关系的数据图，而概念图没有

### Build a Physical Model

workBench的一些基操。

## Primary Keys

easy

## Foreign Keys

有两种情况，比如注册关系。

1. 添加属性注册id为注册关系的主键。

   好处：如果注册关系表和别的表存在父子关系，那么在添加别的表外键时，只用注册id即可。

   坏处：对于数据合理性缺乏天然检验，比如可能重复添加了相同的数据。

2. 设置学生id和课程id为复合主键。

   好处：存在天然校验，重复添加相同的课程会失败

   坏处：如果注册关系表和别的表存在父子关系，那么在添加别的表外键时，需要添加复合主键属性，比较复杂。

## Foreign Key Constraints

外键约束有四种，父表中的对应记录被删除或更新时的该表的操作。

- ### RESTRICT

  拒绝更新，删除

- ### CASCADE

  当学生id从1变成2时，该注册表中的student_id 也从1变成2。删除掉1时，该表也全删掉，

- ### SET NULL

  置为null，会变成孤儿记录

- ### NO ACTION

  什么也不做，也是拒绝操作。

## Normalization

标准化是审查我们的设计，并确保它遵循一些防止数据重复的预定义规则的这一过程。

基本上7条规则，也被称为七范式。

99%的应用场景，只需要应用前3条范式。

### First Normal Form  (1NF)

Each cell should have a single value and we cannot have repeated columns.

(一行中每个单元格都应该有单一值，且不能出现重复列)

这就让之前courses表中的tags必须分出来，新建一个tag表。

### Link Tables

1:N  先点子表再点父表。

### Second Normal Form（2NF）

要满足第二范式，那么一定要先满足第一范式。

Each table should describe one entity，and every column in that table should describe that entity。

### Third Normal Form（3NF)

必须满足第二范式。

A column in a table should not be derived from other columns。

first_name last_name 那么就应该再有full name.

## My Pragmatic Advice

专注消除冗余，先从逻辑或概念模型开始，不要直接跳到创建表。

## Don't Model the Universe

SOLVE TODAY PROBLEM.

Build a model for your problem domain, not the real world.

## Forward Engineering a Model

find Database and select Forward Engineer.

## Synchronizing a Model

没有数据库的时候使用正向工程，有数据库时候使用同步模型。

## Reverse Engineering a Database

如何修改一个没有模型的数据库？select database and select reverse Engineering

## Creating Databases

```SQL
-- 创建数据库
CREATE DATABASE IF NOT EXISTS sql_store2;
-- 删除数据库
DROP DATABASE IF EXISTS sql_store2;
```

## Creating Tables

```SQL
CREATE DATABASE IF NOT EXISTS sql_store2;
USE sql_store2;
-- DROP TABLE IF EXISTS customers; 两种创建判断
CREATE TABLE IF NOT EXISTS customers -- 同上
(
	customer_id INT PRIMARY KEY AUTO_INCREMENT,	-- 自动递增
    first_name	VARCHAR(50) NOT NULL,			-- 非空
    points  		INT NOT NULL DEFAULT 0,		-- 默认为0
    email			VARCHAR(255) NOT NULL UNIQUE-- 唯一
);
```

## Altering Tables

```sql
ALTER TABLE customers
		ADD last_name VARCHAR(50) NOT NULL AFTER first_name, -- 'first name' 如果属性有空格，则加上引号。
        ADD city 		  VARCHAR(50) NOT NULL,
        MODIFY COLUMN first_name VARCHAR(55) DEFAULT '',
        DROP points
        ;
```

## Creating Relationships

```SQL
CREATE DATABASE IF NOT EXISTS sql_store2;
USE sql_store2;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS customers;
CREATE TABLE IF NOT EXISTS customers
(
	customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name	VARCHAR(50) NOT NULL,
    points  		INT NOT NULL DEFAULT 0,
    email			VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE orders
(
		order_id INT PRIMARY KEY,
        customer_id INT NOT NULL,
        FOREIGN KEY fk_orders_customers (customer_id) -- fk__外键的名字，customer_id 外键在这里属性显示的名字
			REFERENCES customers (customer_id) -- 指明是依据哪个主键的外键
            ON UPDATE CASCADE
            ON DELETE NO ACTION
);
```

## Altering Primary/Foreign Keys

```SQL
ALTER TABLE orders
		ADD PRIMARY KEY(order_id), 			-- 添加主键，如果有多个主键，用逗号分隔
        DROP PRIMARY KEY,					-- 去掉主键不需要加别的。
		DROP FOREIGN KEY orders_ibfk_1,
        ADD FOREIGN KEY fk_orders_customers (customer)
				REFERENCES customers (customer_id)
				ON UPDATE CASCADE
				ON DELETE NO ACTION;
```

## Character Sets and Collations

字符集是将字符转换为数字，字符集有好几种

`SHOW CHARSET;`可以看所有支持的字符集。

`CHAR(10)` 10指的是字符

如果想更改字符集，右键数据库，select Schema Inspector，可以看到数据库的字符集。

鼠标只能改表和列的字符集，

选中表，列 就可以改了。

```sql
-- 创建时更改
CREATE DATABASE db_name
		CHARACTER SET latin1
-- 现有的更改
ALTER DATABASE db_name	
		CHARACTER SET latin1
-- 创建时改表字符集      
CREATE TABLE table1
(
)
CHARACTER SET latin1

-- 修改时改表字符集
ALTER TABLE table1
(
)
CHARACTER SET latin1
```

## Storage Engines

MyISAM 

InnoDB(支持事务的牛牛引擎)

存储引擎是表级别的。

```sql
ALTER TABLE customers
ENGINE = InnoDB
```































































