---
title: "MySQL的explain解释器探索"
date: "2018-12-04"
categories:
  - MySQL
toc: true
---

在SQL的执行前，会对SQL语句进行拆解分析，通过在SQL语句前加`explain`能将这一信息提现出来，且不进行真正的查询。

索引是SQL语句执行的重中之重，作为程序员必须要足够了解。怎么建立索引？走哪个索引？为什么？...


### 一、字段基本解读

```sql
+-----+-------------+-------+------+---------------+------------+---------------+-----------+------+-------+
| id  | select_type | table | type | possible_keys |    key     |    key_len    |    ref    | rows | Extra |
+-----+-------------+-------+------+---------------+------------+---------------+-----------+------+-------+
| 编号 |   查询类型   |  表   |  类型 |  预测用到的索引  | 实际使用索引 | 实际使用索引长度 | 表之间引用 | 行数 | 额外信息 |
+-----+-------------+-------+------+---------------+------------+---------------+-----------+------+-------+
```
一般来说关注``type``,``key``,``key_len``,``Extra``段进行SQL优化即可。
### 二、创建例表
```sql
#创建用户表
CREATE TABLE `users`(
	`id` INT UNSIGNED AUTO_INCREMENT,
	`name` VARCHAR(8) NOT NULL DEFAULT '',
	`age` TINYINT NOT NULL DEFAULT 0,
	`email` VARCHAR(32),
	`info` VARCHAR(255) COMMENT '无索引',
	PRIMARY KEY(id) COMMENT '主键索引',
	UNIQUE INDEX ui_email(email) COMMENT '唯一索引',
	INDEX i_age(age) COMMENT '普通索引',
	INDEX i_name_age(name,age) COMMENT '复合索引'
)ENGINE = INNODB DEFAULT CHARSET = utf8;
#插入一条
INSERT INTO `users`(`name`,`age`,`email`,`info`)
	VALUES('voyager',24,'772532526@qq.com','人生若只如初见');
#创建卡片表，用户与卡片一对多
CREATE TABLE `card`(
	`cid` INT UNSIGNED AUTO_INCREMENT,
	`uid` INT NOT NULL DEFAULT 0,
	`card` VARCHAR(255) NOT NULL DEFAULT '',
	PRIMARY KEY(`cid`)
	#暂不为uid加索引
)ENGINE = InnoDB DEFAULT CHARSET = UTF8;
#插入两条数据
INSERT INTO `card`(`uid`,`card`)
	VALUES(1,'何事秋风悲画扇'),(1,'等闲变却故人心，却道故人心易变');
```
### 三、单独释义
**1. id值**
> 标记分解的sql执行顺序，非直观的从上至下的顺序。
> id值相同，从上至下执行。
> id值不同，从大到小执行。

以此结论，反观实例，仅观察 id 和 table 部分。
```sql
#SQL1
mysql> EXPLAIN SELECT * FROM users,card WHERE id = uid AND cid = 1;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | card  | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
|  1 | SIMPLE      | users | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
```
SQL1，虽然以``cid``为条件置于后边，但因其为主键，SQL解释器知道使用它先筛选更快。
```sql
#SQL2
mysql> EXPLAIN SELECT * FROM users,card WHERE uid = 1 AND id = uid ;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra       |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------------+
|  1 | SIMPLE      | users | const | PRIMARY       | PRIMARY | 4       | const |    1 |             |
|  1 | SIMPLE      | card  | ALL   | NULL          | NULL    | NULL    | NULL  |    2 | Using where |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------------+
```
SQL2，``uid``作为条件，却先筛选``users``表，原因为：``作为条件的字段非索引，则先筛选数据量小的表``。
```sql
#SQL3
mysql> EXPLAIN SELECT * FROM users WHERE id =
			(SELECT uid FROM card WHERE cid = 2);
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | PRIMARY     | users | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
|  2 | SUBQUERY    | card  | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
```
SQL3，加入一个子查询，id不同，先进行2子查询，再进行1主查询。
<br>
**2.select_type查询类型**
|类型|出现情况|
|:--|:--|
SIMPLE|简单查询，不包含子查询、UNION，例SQL1
PRIMARY|复杂查询中的主查询，例SQL3
UBQUERY|包含子查询语句的子查询可能出现，例SQL3
UNION|联合查询，t1 UNION t2 中的t2
UNION RESULT| 联合结果
DERIVED|衍生查询，一般为产生的临时表
举例一个UNION的
```sql
#SQL4
mysql> EXPLAIN SELECT uid FROM card WHERE cid = 1
				UNION
				SELECT uid FROM card WHERE cid = 2;
+------+--------------+------------+-------+---------------+---------+---------+-------+------+-------+
| id   | select_type  | table      | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+------+--------------+------------+-------+---------------+---------+---------+-------+------+-------+
|    1 | PRIMARY      | card       | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
|    2 | UNION        | card       | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
| NULL | UNION RESULT | <union1,2> | ALL   | NULL          | NULL    | NULL    | NULL  | NULL |       |
+------+--------------+------------+-------+---------------+---------+---------+-------+------+-------+
```
还有一些其他的，不详细列出。
<br>
**3.table表**
一般指查询的表，对于带尖括号的，表示``select_type + id``的指向。例如SQL4，``<union1,2>``表示id为1和2联合出来的表。
<br>

**4.type索引类型**
速度``null > syetem > const > eq_ref > ref > range > index > all``
|类型|出现情况|
|:--|:--|
null|甚至不需要访问索引表，例如主键作为条件超过当前表主键最大值;
system|const的特殊情况，只有一条数据的系统表，或衍生表只有一条数据的主查询？？
const|使用唯一索引等价查询，仅能匹配到一条数据
eq_ref|使用唯一索引作为关联条件，匹配多条不重复数据
ref| 普通索引等价
range|检索给定范围的索引 ， > 、< 、>= 、<=、between  and
index|仅查询索引表
all|遍历全表以找到匹配的行
例子在后边。
<br>
**5.posibble_keys预测用到的索引**
写sql语句时，有些看起来会使用的索引，但实际可能抛弃索引，遍历全表。

<br>

**6.keys实际使用的索引**

<br>

**7.key_lengths实际使用的索引长度**
>一般用于判断复合索引是否被完全使用。
>utf8中一个字符占3个字节。
>null占1个字节。
>可变长度占2个字节。
```sql
#SQL5
mysql> EXPLAIN SELECT * FROM users WHERE name = 'a' AND age = 2;
+----+-------------+-------+------+------------------+-------+---------+-------+------+-------------+
| id | select_type | table | type | possible_keys    | key   | key_len | ref   | rows | Extra       |
+----+-------------+-------+------+------------------+-------+---------+-------+------+-------------+
|  1 | SIMPLE      | users | ref  | i_age,i_name_age | i_age | 1       | const |    1 | Using where |
+----+-------------+-------+------+------------------+-------+---------+-------+------+-------------+
```
SQL5，我本意是想用到联合索引``i_name_age``，但是实际上被索引``i_age``干扰了，这也解释了<kbd>possible_keys</kbd>和<kbd>keys</kbd>的不同，而``i_age``作为``tinyint``占1个字符，所以<kbd>key_len</kbd>值为1。好的，这种时候需要考虑``i_age``的必要性，要么删掉此索引，要么强制使用索引。
这里我选择了先删除在添加回来，却发现了一个有意思的事情。
```sql
mysql> ALTER TABLE users DROP INDEX i_age;
#SQL6
mysql> EXPLAIN SELECT * FROM users WHERE name = 'a' AND age = 2;
+----+-------------+-------+-------+---------------+------------+---------+------+------+-----------------------+
| id | select_type | table | type  | possible_keys | key        | key_len | ref  | rows | Extra                 |
+----+-------------+-------+-------+---------------+------------+---------+------+------+-----------------------+
|  1 | SIMPLE      | users | range | i_name_age    | i_name_age | 27      | NULL |    1 | Using index condition |
+----+-------------+-------+-------+---------------+------------+---------+------+------+-----------------------+
```
没问题，``name``为``varchar(8) not null``，``8 * 3 + 2 + 1 = 27``。
然后为了测试把``i_age``加回来。
```sql
mysql> ALTER TABLE users ADD INDEX i_age(`age`);
#SQL7
mysql> EXPLAIN SELECT * FROM users WHERE name = 'a' AND age = 2;
+----+-------------+-------+------+------------------+------------+---------+-------------+------+-----------------------+
| id | select_type | table | type | possible_keys    | key        | key_len | ref         | rows | Extra                 |
+----+-------------+-------+------+------------------+------------+---------+-------------+------+-----------------------+
|  1 | SIMPLE      | users | ref  | i_name_age,i_age | i_name_age | 27      | const,const |    1 | Using index condition |
+----+-------------+-------+------+------------------+------------+---------+-------------+------+-----------------------+
```
哎？发现这回``i_age``并没有干扰到复合索引的使用。我猜测与索引建立的顺序有关，观察两次``possible_keys``索引顺序，并删除再添加回来``i_name_age``情况又和SQL5一样，也验证了这一点。
>注意复合索引与普通索引的关系及创建顺序。

<br>

**8.ref**
表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

<br>

**9.rows**
根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数

<br>

**10.extra**

|名词|解释|
|:--|:--|
Using index|性能提升，索引覆盖，此查询仅查询索引不需要回表查询
Using where|在查找使用索引的情况下，需要回表去查询所需的数据
Using filesort|性能消耗大，需要额外一次文件排序
Using temporary|性能消耗大，用到临时表，常见于排序和分组查询
Using join buffer|连接缓存
Using where; Using index |查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

### 实例
为了更易懂，添加一列``money``，删除索引``i_name_age``和``i_age``，创建``i_money_name_age``。
```sql
ALTER TABLE users ADD COLUMN money INT NOT NULL DEFAULT 10;
ALTER TABLE DROP INDEX i_age;
ALTER TABLE DROP INDEX i_name_age;
ALTER TABLE users ADD INDEX i_money_name_age(money,name,age);
```
1. 复合索引使用必须从左开始，且不能跨列，不能计算。
	```sql
	#SQL8
	mysql> EXPLAIN SELECT * FROM users WHERE age = 12 AND name = '12' AND money = 12;
	+----+-------------+-------+------+------------------+------------------+---------+-------------------+------+-----------------------+
	| id | select_type | table | type | possible_keys    | key              | key_len | ref               	| rows | Extra                 |
	+----+-------------+-------+------+------------------+------------------+---------+-------------------+------+-----------------------+
	|  1 | SIMPLE      | users | ref  | i_money_name_age | i_money_name_age | 31      | const,const,const |    1 | Using index condition |
	+----+-------------+-------+------+------------------+------------------+---------+-------------------+------+-----------------------+
	```
	SQL8没按顺序但使用了复合索引，是因为SQL优化器的缘故把条件重排序了。
	```sql
	#SQL9
	mysql> EXPLAIN SELECT * FROM users WHERE name = '12' AND age = 12;
	+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
	| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
	+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
	|  1 | SIMPLE      | users | ALL  | NULL          | NULL | NULL    | NULL |    2 | Using where |
	+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
	```
	SQL9没从左开始，都未使用上索引。
	```sql
	#SQL10
	mysql> EXPLAIN SELECT * FROM users WHERE money = 12 AND age = 12;
	+----+-------------+-------+------+------------------+------------------+---------+-------+------+-----------------------+
	| id | select_type | table | type | possible_keys    | key              | key_len | ref   | rows | Extra                 |
	+----+-------------+-------+------+------------------+------------------+---------+-------+------+-----------------------+
	|  1 | SIMPLE      | users | ref  | i_money_name_age | i_money_name_age | 4       | const |    1 | Using index condition |
	+----+-------------+-------+------+------------------+------------------+---------+-------+------+-----------------------+
	```
	SQL10，观察索引长度，因为跨过``name``，所以``age``没有使用上索引。
2. 没有where时，注意order by。
	```sql
	#SQL11
	mysql> EXPLAIN SELECT * FROM users ORDER BY money;
	+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
	| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra          |
	+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
	|  1 | SIMPLE      | users | ALL  | NULL          | NULL | NULL    | NULL |    2 | Using filesort |
	+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
	```
	SQL11，此时select * 导致order by不使用索引。
	```sql
	#SQL12
	mysql> EXPLAIN SELECT id,money,age FROM users ORDER BY money;
	+----+-------------+-------+-------+---------------+------------------+---------+------+------+-------------+
	| id | select_type | table | type  | possible_keys | key              | key_len | ref  | rows | Extra       |
	+----+-------------+-------+-------+---------------+------------------+---------+------+------+-------------+
	|  1 | SIMPLE      | users | index | NULL          | i_money_name_age | 31      | NULL |    2 | Using index |
	+----+-------------+-------+-------+---------------+------------------+---------+------+------+-------------+
	```
	SQL12，如果选中复合索引里包含的字段和主键，会使用索引。
3. 注意索引建立的排序方式
	```sql
	#SQL13
	mysql> EXPLAIN SELECT id,money,age FROM users ORDER BY money,name,age DESC;
	+----+-------------+-------+-------+---------------+------------------+---------+------+------+-----------------------------+
	| id | select_type | table | type  | possible_keys | key              | key_len | ref  | rows | Extra                       |
	+----+-------------+-------+-------+---------------+------------------+---------+------+------+-----------------------------+
	|  1 | SIMPLE      | users | index | NULL          | i_money_name_age | 31      | NULL |    2 | Using index; Using filesort |
	+----+-------------+-------+-------+---------------+------------------+---------+------+------+-----------------------------+
	```
	SQL13，这里产生了``Using filesort ``，因为默认索引建立顺序都是``ASC``，三个字段顺序必须一致。

### 关于order by使用索引
在这里插入图片描述
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528193933112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o3NzI1MzI1MjY=,size_16,color_FFFFFF,t_70)
### 疑惑
```sql
#SQL9
mysql> EXPLAIN SELECT * FROM users WHERE name = 'voyager' AND age > 12;
+----+-------------+-------+-------+-----------------------------+------------+---------+------+------+-----------------------+
| id | select_type | table | type  | possible_keys               | key        | key_len | ref  | rows | Extra                 |
+----+-------------+-------+-------+-----------------------------+------------+---------+------+------+-----------------------+
|  1 | SIMPLE      | users | range | i_name_age,i_age,i_age_name | i_name_age | 27      | NULL |    2 | Using index condition |
+----+-------------+-------+-------+-----------------------------+------------+---------+------+------+-----------------------+
1 row in set

mysql> EXPLAIN SELECT name,age FROM users WHERE name = 'voyager' AND age > 12;
+----+-------------+-------+------+-----------------------------+------------+---------+-------+------+--------------------------+
| id | select_type | table | type | possible_keys               | key        | key_len | ref   | rows | Extra                    |
+----+-------------+-------+------+-----------------------------+------------+---------+-------+------+--------------------------+
|  1 | SIMPLE      | users | ref  | i_name_age,i_age,i_age_name | i_name_age | 26      | const |    2 | Using where; Using index |
+----+-------------+-------+------+-----------------------------+------------+---------+-------+------+--------------------------+
1 row in set

mysql> EXPLAIN SELECT name,age FROM users WHERE name = 'voyager' ORDER BY age;
+----+-------------+-------+------+---------------+------------+---------+-------+------+--------------------------+
| id | select_type | table | type | possible_keys | key        | key_len | ref   | rows | Extra                    |
+----+-------------+-------+------+---------------+------------+---------+-------+------+--------------------------+
|  1 | SIMPLE      | users | ref  | i_name_age    | i_name_age | 26      | const |    2 | Using where; Using index |
+----+-------------+-------+------+---------------+------------+---------+-------+------+--------------------------+
```

