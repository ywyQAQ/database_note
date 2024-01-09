## Indexes

Design indexes based on your queries,not your tables.

## Creating Indexes

```sql
EXPLAIN SELECT customer_id FROM customers WHERE state = 'CA';
```

解释了这条语句的查询过程

type : ALL   指的做了一个全表扫描

rows:指扫描的记录条数

```SQL
CREATE INDEX idx_state ON customers(state); -- 创建索引。
EXPLAIN SELECT customer_id FROM customers WHERE state = 'CA';-- 再执行一遍
```

type 变成了ref ，rows也不再是1011

possible_keys 指的是为了执行这个查询可能会考虑的索引

key展示的是实际使用的索引。



## Viewing Indexes

```sql
SHOW INDEXES IN customers
```

第一个是主键，也被称为聚集索引。每张表最多一个聚集索引。

Collation指数据在索引中的排序，A是升序，D是降序。

Cardinality 表示索引中唯一值的估计数量

我这里的idx_state 和 idx_points 是 二级索引，建立索引时，MYSQL自动将id或主键列纳入二级索引。

MySQL 会自动为外键创建索引，方便快速连接表。

## Prefix Indexes

```sql
-- 调参。求出较大的唯一值。
SELECT
		COUNT(DISTINCT LEFT(last_name,1)),
        COUNT(DISTINCT LEFT(last_name,5)),
        COUNT(DISTINCT LEFT(last_name,10))
FROM customers;

CREATE INDEX idx_lastname ON customers (last_name(5))
-- 建立在字符串上的索引
```

所以，5是最佳的前缀长度。

## Full - text Indexes

全文索引可以在应用程序中打造快速强大的搜索引擎，它们包括整个字符串列，而不只是存储前缀，他们会护士任何停止词，比如: in on the 什么的，本质上，他们存储了一套单词列表，对于每个单词，它们又存储了一列这些单词会出现的行或记录。

```sql
CREATE FULLTEXT INDEX idx_title_body ON posts (title,body);

SELECT *
FROM posts
WHERE MATCH(title,body) AGAINST('react redux');
```

还有一个优点，就是包含了 相关性得分，MYSQL 会基于若干因素，为包含了搜索短语的每一行计算相关性得分，相关性得分是一个介于0到1的浮点数，0表示没有相关性，1代表完全相关

```sql
CREATE FULLTEXT INDEX idx_title_body ON posts (title,body);

SELECT *,MATCH(title,body) AGAINST('react redux') -- Match后面是相关性得分列。
FROM posts
WHERE MATCH(title,body) AGAINST('react redux');
```

全文搜索有两种模式。一个是自然语言模式，是默认情况的模式。也就是现在用的。

另一种是布尔模式，这个模式可以包括或排除某些单词，

```SQL
-- 寻找有react 但是不包括redux的行。
SELECT *
FROM posts
WHERE MATCH(title,body) AGAINST('react -redux'IN BOOLEAN MODE) ;

-- 寻找有react 但是不包括redux，且必须有form的行。
SELECT *
FROM posts
WHERE MATCH(title,body) AGAINST('react -redux +form'IN BOOLEAN MODE) ;

-- 寻找有固定短语“handling a form”的行。
SELECT *
FROM posts
WHERE MATCH(title,body) AGAINST('“handling a form”'IN BOOLEAN MODE) ;

```

## Composite Indexes

```SQL
EXPLAIN
SELECT customer_id FROM customers
WHERE state = 'CA' and points > 1000;
```

type 为 ref 

possible_keys 是 idex_state,idx_points.

key 是 idex_state

rows 是113

无论有多少索引，MYSQL最多只会选1个索引。

先使用州索引，来快速找到位于加州的顾客，但之后只能表查询了。

复合索引允许对多列建立索引

```sql
-- 复合索引
CREATE INDEX idx_state_points ON customers(state,points); -- 顺序也有说法
EXPLAIN
SELECT customer_id FROM customers
WHERE state = 'CA' and points > 1000;
```

type 是 range

possible_keys 是idx_state,idx_points,idx_state_points

key 是 idx_state_points

row是58

索引很多如果不能改善性能，每次更新数据要更新索引，会影响效率。

MYSQL 一个索引最多可以包含16列。4到6列 之间达到很好的性能。

## Order of Columns in Composite indexes

**Two Basic Rules**

第一条规则，应该对列进行排序。让更频繁使用的列排在最前面。

第二条规则，应该把基数更高的列排在前面，基数是指索引确定的唯一值的个数，比如州和性别，就应该让州在前面。

但这个规则，真的不好说，实际上用什么还是要看数据库的样子。

比如州和姓名，但实际上根据数据库，虽然姓名唯一值更多，但是有时候也以州在前面更快。

还有强约束什么的在前面更好一点。

只优化性能关键的查询。

考虑查询本身！

## When Indexes are Ignored

```sql
EXPLAIN SELECT customer_id FROM customers
WHERE state = 'CA' OR points > 1000;
```

type 是 index，比表扫描快，因为不涉及从磁盘读取每个记录

rows 是 1011

这可以分为两个子查询。

第一个先找在CA的人，第二个是找超过1000积分的人查询，两个进行一个UNION.

```sql
EXPLAIN SELECT customer_id FROM customers
WHERE points + 10 > 2010;
-- 这种使用了列的，mysql就不能以最优方式利用索引了。
```

## Using Indexes for Sorting

```sql
EXPLAIN SELECT customer_id FROM customers
ORDER BY state;
```

Extra：Using index。因为state 有索引。

```sql
EXPLAIN SELECT customer_id FROM customers
ORDER BY first_name;
```

Extra：using filesort。因为first_name没有索引。外部排序很耗费时间。

通常来说，除非真的必要。

Order by后面的属性顺序很重要！ 

假如 a，b 有索引

1. 可以按a排序
2. 可以按a，b排序
3. 可以按同样的列降序排序

a，c，b会全表扫描。

b 会 Using Index and  filesort

```sql
-- 因为我们有idx_state_points ，州里面已经根据point排过序了，所以这个就很快，
EXPLAIN SELECT customer_id FROM customers
WHERE state =  'CA'
ORDER BY points DESC;

SHOW STATUS LIKE 'last_query_cost';
```

## Covering Indexes

idx_state_point 复合索引包含了关于每个顾客的三条信息，customer_id , state, point

```sql
EXPLAIN SELECT customer_id,state FROM customers
WHERE state =  'CA'
ORDER BY points DESC;

SHOW STATUS LIKE 'last_query_cost';
```

这就叫“覆盖索引”：一个包含所有满足查询需要的数据的索引。通过使用这个素银，Mysql可以在不读取表的情况下就执行查询。

## Index Maintennance

### Duplicate Indexes

同一组列上且顺序一致的索引，ABC and ABC，创建新索引前检查一下，不要创建重复的。

### Redundant Indexes

假如 A 和 B上有一个索引，然后在列A上创建另一个索引。

（A,B） 可以优化 （A）

***Before creating new indexes，check the existing ones。***



























