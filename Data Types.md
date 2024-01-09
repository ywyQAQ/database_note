## Introduction

## String Types

most common char and varchar

| 类型       | 长度             | 备注                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| CHAR       | fix-length       | 固定长度                                                     |
| VARCHAR    | 65535 characters | 最常用，可被编入索引，短一点的string 用VARCHAR(50)，中等长度的用VARCHAR(255) |
| MEDIUMTEXT | 16MB             |                                                              |
| LONGTEXT   | 4GB              |                                                              |

上面的比较常用

| 类型     | 长度      | 备注                    |
| -------- | --------- | ----------------------- |
| TINYTEXT | 255 bytes | 这两个最好还是用varchar |
| TEXT     | 64 kB     | 这两个最好还是用varchar |



- **English 1 bytes 	**
- **European Middle-eastern 2-bytes	**
- **Asian 3 bytes**

## Integer Types

#### TYPES

| 类型             | 长度          | 备注 |
| ---------------- | ------------- | ---- |
| TINYINT          | 1b [-128,127] |      |
| UNSIGNED TINYINT | [0,255]       |      |
| SMALLINT         | 2b [-32k,32k] |      |
| MEDIUMINT        | 3b [-8M,8M]   |      |
| INT              | 4b [-2B,2B]   |      |
| BIGINT           | 8b [-9Z,9Z]   |      |

#### ZEROFULL

INT(4) :      1 => 0001

Use the smallest data type thar suits your needs.

## Fixed-point and Floating-point Types

three types

| 类型         | 长度 | 备注                               |
| ------------ | ---- | ---------------------------------- |
| DECIMAL(p,s) |      | p:精度(一共几位数字) s:小数位数    |
| FLOAT        | 4b   | 用于科学计算，计算很大或很小的数字 |
| DOUBLE       | 8b   | 用于科学计算，计算很大或很小的数字 |

PS:DEC,NUMERIC,FIXED 都是指的DECIMAL

## Boolean Types

BOOL BOOLEAN EQUALS TO TINYINT 

TRUE AS 1

FALSE AS 0

## Enum and Set Types

sometimes we want to limit a colomn to a restricted range,so using Enum Types.

```sql
ENUM('s','x','xl','l')
SET(....)
-- SET那一列可以不止一个
-- Not useful
```

## Date and Times Types

| 类型      | 备注                                                         |
| --------- | ------------------------------------------------------------ |
| DATE      | 只存日期                                                     |
| TIME      | 只存时间                                                     |
| DATETIME  | 日期和时间                                                   |
| TIMESTAMP | 时间戳，是四个字节的，只能存储到20328年之前的日期（被称为2038问题） |
| YEAR      | 四位数的年                                                   |

## Blob Type

二进制长对象来存储大型二进制数据。图像，视频，pdf，word

| 类型       | 备注                     |
| ---------- | ------------------------ |
| TINYBLOB   | 最大存储255b的二进制数据 |
| BLOB       | 65KB                     |
| MEDIUMBLOB | 16MB                     |
| LONGBLOB   | 4GB                      |
|            |                          |

## JSON Type

很多种json表示方法。

第一种

```sql
UPDATE products
SET properties = '
{
	"dimensions":[1,2,3],
    "weight":10,
    "manufacture":{"name":"sony"}
}
'
WHERE product_id = 1;
```

第二种

```sql
UPDATE products
SET properties = JSON_OBJECT(
		'weight',10,
        'dimensions',JSON_ARRAY(1,2,3),
        'manufacturer',JSON_OBJECT('name','sony')
)
WHERE product_id = 1;
```

```sql
SELECT product_id,JSON_EXTRACT(properties,'$.weight')
FROM products
WHERE product_id =1;
-- 这样可以提取属性

SELECT product_id,JSON_EXTRACT(properties,'$.dimensions[0]') -- 1
FROM products
WHERE product_id =1;
-- 这样可以提取属性

SELECT product_id，properties->'$.dimensions[0]') -- 1
FROM products
WHERE product_id =1;
-- 也可以这样可以提取属性

SELECT product_id，properties->'$.manufacturer.name' -- 1
FROM products
WHERE product_id =1;
-- 也可以用名字,提取出的字符串往往有双引号，可以将-> 换成->> 去掉
```

也可以单独修改json里的某项值，更新现有属性或添加新属性。

```sql
UPDATE products
SET properties = JSON_SET(
		properties,
        '$.weight',20,
        '$.age',10
)
WHERE product_id = 1;
```

也可以删除属性

```sql
UPDATE products
SET properties = JSON_REMOVE(
		properties,
        '$.age'
)
WHERE product_id = 1;
```







