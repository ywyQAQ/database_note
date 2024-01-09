## Creating a User

```sql
-- @后接用户的连接位置。
CREATE USER john@127.0.0.1 
CREATE USER john@localhost
CREATE USER john@codewithmosh.com -- 可以在这个域里任何一个计算机连接，但是无法从codewithmosh.com的子网连接。
CREATE USER john@'%.codewithmosh.com' -- 可以在这个域里任何一个计算机连接,也可从codewithmosh.com的子网连接。

CREATE USER john IDENTIFIED BY '1234' -- 密码
```

## Viewing Users

```SQL
SELECT * FROM mysql.user;
```

Host 列表示的是 可连接的位置。

## Dropping Users

```sql
DROP USER bob@codewithmosh.com
```

## Changing Passwords

```sql
SET PASSWORD FOR john = '1234'; -- 帮john改
SET PASSWORD = '1234'			-- 帮当前用户改
```

## Granting Privileges

```sql
-- 1:web/desktop application

CREATE USER moon_app IDENTIFIED BY '1234';

GRANT SELECT,INSERT,UPDATE,DELETE,EXECUTE -- 执行存储过程
ON sql_store.* -- sql_store 所有表
-- ON sql_store.customers
TO moon_app;
```

```sql
-- 2:admin
CREATE USER john IDENTIFIED BY '1234';
GRANT ALL 
ON *.* -- 所有数据库所有表
-- ON sql_store.*
TO john；
```

## Viewing Privileges

```sql
SHOW GRANTS FOR john;

SHOW GRANTS -- 查看当前用户权限。
```

## Dropping Privileges

```sql
REVOKE CREATE VIEW
ON sql_store.*
FROM moon_app;
```



































