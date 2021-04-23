# MySQL学习

[toc]

### 教程

以下为对MySQL5.7官方文档中的**tutorial**章节的重要点的记录：

#### Entering Queries

```bash
mysql> select version(), current_date(), now(); #大小写均可，now()的结果精确到时分秒
# \c，取消命令
```

#### Creating and Using a Database

```bash
#授予指定用户在指定主机上对menagerie数据库中的所有表的所有操作权限
mysql> GRANT ALL ON menagerie.* TO 'your_mysql_name'@'your_client_host';
```

##### Loading Data into a Table

```bash
Whistler        Gwen    bird    \N      1997-12-09      \N

# 加载文件pet.txt中的数据到表pet中，pet.txt中的数据格式如上所示，注意NULL用\N表示
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;

# 在windows系统中使用以'\r\n'为行结束符的编辑器创建pet.txt时，应该这样加载数据
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet
       LINES TERMINATED BY '\r\n';
```

##### Date Calculations

```bash
# curdate() = current_date()
mysql> SELECT name, birth, CURDATE(),
       TIMESTAMPDIFF(YEAR,birth,CURDATE()) AS age
       FROM pet;
       
# 查询条件为：月份=当前日期的下一个月的月份
mysql> SELECT name, birth FROM pet
       WHERE MONTH(birth) = MONTH(DATE_ADD(CURDATE(),INTERVAL 1 MONTH));
mysql> SELECT name, birth FROM pet
       WHERE MONTH(birth) = MOD(MONTH(CURDATE()), 12) + 1;
```

##### Working with NULL Values

```bash
# NULL是没有数据，不等于0或者''
mysql> SELECT 0 IS NULL, 0 IS NOT NULL, '' IS NULL, '' IS NOT NULL;
+-----------+---------------+------------+----------------+
| 0 IS NULL | 0 IS NOT NULL | '' IS NULL | '' IS NOT NULL |
+-----------+---------------+------------+----------------+
|         0 |             1 |          0 |              1 |
+-----------+---------------+------------+----------------+
```

##### Pattern Matching

- 方式一：标准SQL模式匹配

```bash
# 匹配以b开头的（不区分大小写），如果是lile binary 'b%'，则区分大小写
mysql> SELECT * FROM pet WHERE name LIKE 'b%'; 
mysql> SELECT * FROM pet WHERE name LIKE '%fy';
mysql> SELECT * FROM pet WHERE name LIKE '%w%';
mysql> SELECT * FROM pet WHERE name LIKE '_____'; #匹配name长度为5的
```

- 方式二：基于扩展正则表达式的模式匹配

这种方式只要模式在字符串/值的任何位置匹配，就算匹配成功；而第一种方式需要模式与字符串/值完全匹配，才算匹配成功。

```bash
# 匹配name以b开头的记录行，不区分大小写
mysql> SELECT * FROM pet WHERE name REGEXP '^b';
# 区分大小写
mysql> SELECT * FROM pet WHERE name REGEXP BINARY '^b';
# 匹配name以fy结尾的记录行，不区分大小写
mysql> SELECT * FROM pet WHERE name REGEXP 'fy$';
# 匹配name中含有w字符的记录行，不区分大小写
mysql> SELECT * FROM pet WHERE name REGEXP 'w';
# 匹配name的长度等于5的记录行
mysql> SELECT * FROM pet WHERE name REGEXP '^.....$';
# 匹配name的长度等于5的记录行
mysql> SELECT * FROM pet WHERE name REGEXP '^.{5}$';
```

##### Counting Rows

当查询结果除了`count(*)`之外，还有其他字段时，必须有一个`group by`从句，`group by`后紧跟查询结果中的其他字段。否则，以下情况会发生：

- 如果`sql mode`设置为`only_full_group_by`，一个错误会发生：

```bash
mysql> SET sql_mode = 'ONLY_FULL_GROUP_BY';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT owner, COUNT(*) FROM pet;
ERROR 1140 (42000): In aggregated query without GROUP BY, expression
#1 of SELECT list contains nonaggregated column 'menagerie.pet.owner';
this is incompatible with sql_mode=only_full_group_by
```

- 如果`sql mode`没有设置，处理查询命令时，将每一行视作一个独立的组，但是对每一个命名字段选取的值是不确定的，服务器会从所有行中任意选取一行中的值：

```bash
mysql> SET sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT owner, COUNT(*) FROM pet;
+--------+----------+
| owner  | COUNT(*) |
+--------+----------+
| Harold |        8 |
+--------+----------+
1 row in set (0.00 sec)
```

#### Using mysql in Batch Mode

[参考这一小节](https://dev.mysql.com/doc/refman/5.7/en/batch-mode.html)

#### Examples of Common Queries

##### Using User-Defined Variables

```bash
mysql> SELECT @min_price:=MIN(price),@max_price:=MAX(price) FROM shop;
mysql> SELECT * FROM shop WHERE price=@min_price OR price=@max_price;
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0003 | D      |  1.25 |
|    0004 | D      | 19.95 |
+---------+--------+-------+
```

##### Using Foreign Keys

在使用MyISAM引擎时，实际不能使用外键，只能使用`references`关键字去描述使用哪个外表中的哪个字段充当外键的功能。

```bash
CREATE TABLE person (
    id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
    name CHAR(60) NOT NULL,
    PRIMARY KEY (id)
);

# 以下使用了references关键字描述表shirt的owner字段‘映射’表person中的id字段，充当外键的功能
CREATE TABLE shirt (
    id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
    style ENUM('t-shirt', 'polo', 'dress') NOT NULL,
    color ENUM('red', 'blue', 'orange', 'white', 'black') NOT NULL,
    owner SMALLINT UNSIGNED NOT NULL REFERENCES person(id),
    PRIMARY KEY (id)
) ENGINE=MyISAM;

# 当以上面这种方式创建表时，references关键字不会出现在show create table或者describe的输出结果中
SHOW CREATE TABLE shirt\G
*************************** 1. row ***************************
Table: shirt
Create Table: CREATE TABLE `shirt` (
`id` smallint(5) unsigned NOT NULL auto_increment,
`style` enum('t-shirt','polo','dress') NOT NULL,
`color` enum('red','blue','orange','white','black') NOT NULL,
`owner` smallint(5) unsigned NOT NULL,
PRIMARY KEY  (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1
```

##### Searching on Two Keys

```bash
# 这个命令会自动优化输出，不会输出重复的行记录
SELECT field1_index, field2_index FROM test_table
WHERE field1_index = '1' OR  field2_index = '1'
```

##### Calculating Visits Per Day 

```bash
CREATE TABLE t1 (year YEAR, month INT UNSIGNED,
             day INT UNSIGNED);
INSERT INTO t1 VALUES(2000,1,1),(2000,1,20),(2000,1,30),(2000,2,2),
            (2000,2,23),(2000,2,23);

# 查询每年每月中访问一个网页的天数（一天访问多次算访问一次），注意这里位函数的使用
SELECT year,month,BIT_COUNT(BIT_OR(1<<day)) AS days FROM t1
       GROUP BY year,month;
+------+-------+------+
| year | month | days |
+------+-------+------+
| 2000 |     1 |    3 |
| 2000 |     2 |    2 |
+------+-------+------+
```

##### Using AUTO_INCREMENT

```bash
CREATE TABLE animals (
     id MEDIUMINT NOT NULL AUTO_INCREMENT,
     name CHAR(30) NOT NULL,
     PRIMARY KEY (id)
);

INSERT INTO animals (name) VALUES
    ('dog'),('cat'),('penguin'),
    ('lax'),('whale'),('ostrich');

SELECT * FROM animals;
+----+---------+
| id | name    |
+----+---------+
|  1 | dog     |
|  2 | cat     |
|  3 | penguin |
|  4 | lax     |
|  5 | whale   |
|  6 | ostrich |
+----+---------+

# 可以指定id的值，那么插入的值就是指定的值而不是应该自动增加的下一个值；可以指定id为NULL，即使id设置为NOT NULL，此时插入的值仍然为自动增加的下一个值；每一次自动增加的下一个值从当前所有值中的最大值加一
INSERT INTO animals (id,name) VALUES(100,'rabbit');
INSERT INTO animals (id,name) VALUES(NULL,'mouse');
SELECT * FROM animals;
+-----+-----------+
| id  | name      |
+-----+-----------+
|   1 | dog       |
|   2 | cat       |
|   3 | penguin   |
|   4 | lax       |
|   5 | whale     |
|   6 | ostrich   |
|   7 | groundhog |
|   8 | squirrel  |
| 100 | rabbit    |
| 101 | mouse     |
+-----+-----------+

# 可以设置不从1开始自动增长，使用create table或者alter table设置起始值
mysql> ALTER TABLE tbl AUTO_INCREMENT = 100;
```

- **MyISAM Notes**

```bash
CREATE TABLE animals (
    grp ENUM('fish','mammal','bird') NOT NULL,
    id MEDIUMINT NOT NULL AUTO_INCREMENT,
    name CHAR(30) NOT NULL,
    PRIMARY KEY (grp,id)
) ENGINE=MyISAM;

INSERT INTO animals (grp,name) VALUES
    ('mammal','dog'),('mammal','cat'),
    ('bird','penguin'),('fish','lax'),('mammal','whale'),
    ('bird','ostrich');

SELECT * FROM animals ORDER BY grp,id;
+--------+----+---------+
| grp    | id | name    |
+--------+----+---------+
| fish   |  1 | lax     |
| mammal |  1 | dog     |
| mammal |  2 | cat     |
| mammal |  3 | whale   |
| bird   |  1 | penguin |
| bird   |  2 | ostrich |
+--------+----+---------+
```

- 在以上例子中，当auto_increment列是多列索引一部分时，自动增长列的值在每一组（组成索引的列字段）中独立设置

- 如果auto_increment列同时是多个索引的一部分，那么MySQL使用在auto_increment列开始的索引（如果有的话）来生成序列值。例如，如果animals表包含了主键索引（grp, id）以及索引（id），MySQL将会忽视主键索引来生成序列值，而是使用索引（id）来生成序列值，这将会是一个全表单序列，而不是每组一个序列。