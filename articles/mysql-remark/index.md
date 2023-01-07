---
title: "SQL拆解"
date: "2019-01-10"
categories:
  - MySQL
toc: true
---

<!--more-->

### 一、顺序
SQL语句的顺序不是解析的顺序，实际为
```sql
FROM <left_table>
ON <join_condition>
<join_type> JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT
DISTINCT <select_list>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```
### 二、语句
1. 尽量不使用``SELECT *``
2. 统计使用`` SELECT COUNT(*)``
4. 尽量不使用内置函数，不对字段进行计算
5. 减少``JOIN``，适当字段冗余
6. 适当使用``UNION``代替``OR``条件查询
9. ``WHERE``条件须使用到索引，符合左前缀
10. ``WHERE``条件为非索引查找一条数据时应使用``LIMIT 1``避免找到一条后继续查找
11. ``AND``除索引外的条件，筛选力度大的优先（优化器会自动优化，但是你应该知道）
12. ``LIKE``模糊查询注意符合左前缀
13. ``IN``后数组不宜过大，注意使用``EXISTS``代替
14. ``ORDER BY``适当建立联合索引，注意``SELECT``字段不使用索引问题
15. 注意``OFFSET``过大效率低问题
16. 多使用``EXPLAIN``才是关键

### 三、表
1. 表名必须使用单数小写加下划线形式
2. 对于布尔型字段命名``is_``,``has_``,``can_``等，``unsigned tinyint``，0否1是
3. ``InnoDB``中推荐``varchar``代替``char``  [引文](https://blog.csdn.net/yunhua_lee/article/details/7038780) (但是固定表效率更高？)
4. 小数使用``decimal``存储,``double``、``float``会精度丢失
5. ``text``类型应单独建立表分离出来
6. 主键索引名``pk_``，唯一索引名``uk_``，普通索引名``idx_``
7. 具有唯一性字段必须建立唯一索引
8. 字符型索引的建立一般没必要全长度索引
9. 字段名不该使用``desc``,``range``等保留字
10.  字段尽量``not null``
11. 应该写属性``comment``
12. 实际应用中不使用外键.
13. 一张表只能有一个``AUTO_IN CREMENT``字段且该字段必须为``KEY``

### 四、配置
1. ``max_connections``最大连接数。
2. ``back_log``超过最大连接数存放在堆栈中的等待数
3. ``wait-timeout``连接闲置最大时间值
4. ``thread_concurrency``应设为CPU核数的2倍，cup数*核数\*2
5. ``skip-name-resolve``禁用DNS解析
6. ``key_buffer_size``索引块的缓冲区大小
7. ``innodb_buffer_pool_size``针对InnoDB的参数
8. ``sort_buffer_size``排序缓冲区大小
9. ``set global slow_query_log=1;``开启慢查询日志


