---
title: "binlog开启及数据恢复"
date: "2020-01-05"
categories:
  - MySQL
toc: true
---

binlog的功能这里不再赘述，之前做主从都是在前人的基础上改改配置，突然觉得还是要自己一点点弄一遍才安心。

<!--more-->

#### 一、相关变量查看
1. 是否开启binlog ``show variables like 'log_bin';``
2. binlog三种模式 ``show variables like '%binlog_format%';``


#### 二、配置
1. 配置my.conf
	```conf
	[mysqld]
	server-id = 1
	log-bin = /var/log/mysql/mysql-bin.log #设置log-bin文件自动会开启binlog
	binlog_format = ROW  #格式
	expire-logs-days = 14  #14天内
	max-binlog-size = 500M #每个binlog文件最大值
	```
2. 重启mysql
3. /var/log/mysql/ 文件夹下 mysql-bin.000001 日志文件 和 mysql-bin.index 索引文件，binlog是二进制文件


#### 三、日志SQL操作
1. 查看日志文件列表 ``show master logs;``
2. 产生新日志文件 ``flush logs;``
3. 清除日志文件  ``reset master;``


#### 三、数据测试及备份
备份不是实时的，数据发生错误的恢复会有很多状态
1. 最新备份状态
2. 备份后到错误操作前
3. 错误操作后的用户正常操作

我们需要先恢复备份再通过binlog找回并跳过错误的操作
```sql

create database test_binlog;

use test_binlog;

create table t1 (
	id int unsigned not null auto_increment primary key,
	uname varchar(10) not null default ''
);

insert into t1(uname) values('a');
flush logs;
insert into t1(uname) values('b');
flush logs;
insert into t1(uname) values('c');
flush logs;
# 此时binlog最新为mysql-bin.000004
```

```shell
# 备份数据库 test_binlog，  模拟定时备份
mysqldump -uroot -p test_binlog > /tmp/backup.sql
```

```sql
# 备份后的数据修改
insert into t1(uname) values('d');
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105151221851.png)

```sql
# 啊呀!没带条件把所有数据都改了，这是错误的！
update t1 set uname = '哈哈哈';
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105162636952.png)


```sql
# 用户正常操作-删除
delete from t1 where id = 2;
# 用户正常操作-发现数据怎么不读，我自己改回去吧
update t1 set uname = 'c' where id = 3;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105162655549.png)

#### 五、备份恢复

```sql
1. 系统停止服务 show master status; 得到最新日志 mysql-bin.000004 , position=3099
2. 保护车祸现场，flush logs; 生成新的日志文件 mysql-bin.000005 ， 防止 mysql-bin.000004 被修改
3. 登录SQL，use test_binlog;
4. 恢复备份 source /tmp/backup.sql;
5. 我们测试只有最开始的小a。  如果是正常的话应该是大部分数据都回来了，好开心
6. 接下来就需要通过binlog恢复备份之后的数据了
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105141542953.png)

### 三、binlog恢复
1. 查看日志 ``show binlog events in 'mysql-bin.000004' ;`` 找到回滚的语句
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105162334960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o3NzI1MzI1MjY=,size_16,color_FFFFFF,t_70)
发现这样并不好找，SQL没有暴露出来
2. 提取并翻译binlog  ``mysqlbinlog --no-defaults  --base64-output=decode-rows -v --start-datetime='2021-01-05 15:00:00' --stop-datetime='2021-01-05 15:25:00' /var/log/mysql/mysql-bin.000004 > /tmp/binlog.sql``
	``-v`` 执行
	``--start-datetime``截取开始时间
	``--stop-datetime``截取结束时间
3. 查看 ``/tmp/binlog.sql``
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105161242887.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o3NzI1MzI1MjY=,size_16,color_FFFFFF,t_70)
可以看到插入d的commit的position=570，接下来就是update 哈哈哈了。
我们按照点位恢复数据``mysqlbinlog -v --stop-position=570 /var/log/mysql/mysql-bin.000004 | mysql -u root -p``
此时，我们恢复的状态是截止执行错误SQL前的状态，还需要执行错误SQL之后的SQL
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105162142308.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o3NzI1MzI1MjY=,size_16,color_FFFFFF,t_70)
 执行 ``mysqlbinlog -v --start-position=924 /var/log/mysql/mysql-bin.000004 | mysql -u root -p``
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210105162435103.png)
至此，我们恢复到了相对正确的数据。因为你要不要考虑用户为什么删除第二条呢？ 诸如此类。




