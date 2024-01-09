## Transactions

A group of SQL statements that represent a single unit of work.

所有的语句都应成功完成，否则事务会运行失败。

**ACID**

- ***Atomicity*** 每个事务都是一个工作单元，不管包含多少语句。
- ***Consistency*** 通过事务，数据库始终保持一致的状态。
- ***Isolation*** 事务相互隔离，不会互相干扰
- ***Durability*** 事务的更改是永久的

## Creating Transactions

```SQL
START TRANSACTION;

INSERT INTO orders (customer_id,order_date,status)
VALUES (1,'2019-01-01',1);

INSERT INTO order_items
VALUES (LAST_INSERT_ID(),1,1,1);

COMMIT;
-- ROLLBACK; 可以将操作全部退回。
```

## Concurrency and Locking

```sql
-- 更新同一个表会被锁定。
START TRANSACTION;

UPDATE customers
SET points = points + 10
WHERE customer_id = 1;

COMMIT;
```

## Concurrency Problems

### 1.Lost Update

较晚提交的update可能会覆盖之前的update

### 2.Dirty Reads

事务读取了尚未被提交的数据

所以要为事务之间建立隔离，事务不能读取别的事务刚改过的数据，除非提交了更新。

四个事务隔离等级：后面讲。

### 3.Non-repeating Read

事务不可见外界更改，就算有其他事务更改了数据，我看到的也是初始快照。

### 4.Phantom Reads

数据在执行过程中，突然改变了，就像幽灵一样。

例如第一个事务对一个表中的数据进行了修改，比如这种修改涉及到表中的“全部数据行”。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入“一行新数据”。那么，以后就会发生操作第一个事务的用户发现表中还存在没有修改的数据行，就好象发生了幻觉一样.

## Transaction Isolation Levels

![](F:\幕布代码位置\图片库\Transaction Isolation Level.png)

1. ***Read Comitted*** 解决了 Dirty Read 只能读已提交的数据。但这可能会在事务中会读取同一个内容两遍而得到不同的结果，因为另外一个事务在两次读取之间更新了数据，所以它无法保护我们不进行不可重复读，所以我们才需要可重复读取。
2. ***Repeatable Read*** 确信不同的读取会返回同样的结果，即使数据在这期间有做更改。
3. ***Serializable*** 让事务一个个排队执行。

隔离等级越高，会存在越重的性能和可拓展问题。

隔离等级越低，并发越高，但对应的并发问题也越多。

MYSQL默认事务隔离级别是***REPEATABLE READ***

```SQL
SHOW VARIABLES LIKE 'transaction_isolation';
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- 为下一个事务设置隔离级别。

SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- 所有未来的事务都会是这个隔离级别

SET GLOBAL SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- 所有会话中的所有新事务都会是这个隔离级别
```

存在用时调整，用完恢复这个概念。（修改那个会话或者连接的隔离级别）

## READ UNCOMMITTED Isolation Level

```sql
START TRANSACTION;		-- step 2
UPDATE customers		-- step 3
SET points = 20
WHERE customer_id = 1;
ROLLBACK;
```

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;-- step 1
SELECT points									 -- step 4
FROM customers
WHERE customer_id = 1;
-- 最终会取到20，一个不存在的数据。
```

## READ COMMITED Isolation Level

```SQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;		-- step 1

START  TRANSACTION;									-- step 2
SELECT points FROM customers WHERE customer_id = 1;	-- step 5
SELECT points FROM customers WHERE customer_id = 1;	-- step 8
COMMIT;												-- 可以观察到两次查询数值不一样。
```

```SQL
START TRANSACTION;									-- step 3
UPDATE customers									-- step 4
SET points = 30										-- step 6 把20改成30
WHERE customer_id = 1;
COMMIT;												-- step 7 
```

## REPEATABLE READ Isolation Level

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

START  TRANSACTION;
SELECT * FROM customers WHERE state = 'VA';

COMMIT;
```

```sql
START TRANSACTION;
UPDATE customers
SET state = 'VA'
WHERE customer_id = 1;
COMMIT;
-- 你懂的
```

## SERIALIZABLE Isolation Level

真正消除并发问题。

```SQL
SET TRANSACTION ISOLATION LEVEL SERIALIAZABLE;
```

## Deadlocks

```sql
START  TRANSACTION;
UPDATE customers SET state = 'VA' WHERE customer_id = 1;
UPDATE orders SET status = 1 WHERE order_id = 1;
COMMIT;
```

```sql
START  TRANSACTION;
UPDATE orders SET status = 1 WHERE order_id = 1;
UPDATE customers SET state = 'VA' WHERE customer_id = 1;
COMMIT;
```

互相锁死。死锁一般不是经常发生的事情，如果经常发生，那需要改进系统。我们往往告诉用户，操作失败了，你再试试，死锁往往无法消除，只能降低可能性。

为了减少死锁，我们在更新多条记录的时候，可以遵照相同的顺序。

或者尽量简化事务，缩小事务运行时长。如果实在没法简化事务，那看看能不能安排在非高峰时段运行。





