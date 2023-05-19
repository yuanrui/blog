---
title: MySql SQL语句实现数据表插入或更新
date: 2021-04-27 11:35:00
tags:  
  - Sql
  - MySql
categories:
  - Sql
  - MySql
toc: true
---
### 前言

Spring Data Jpa的增删改查仓库接口(CrudRepository)提供一个sava方法，用于保存实体对象。众所周知，在ORM框架中实体对应数据表，对实体的保存操作最终会转换为sql执行语句。数据保存实际上是执行插入和更新操作，在数据表中不存在该记录时，使用insert语句插入，存在相关记录时，使用update语句更新非主键列。本文主要介绍MySql数据表插入或更新的几种sql语句实现。

在正式开始之前，假设我们要是保存的数据表是*t_sequence*，并且存在部分数据。数据表结构定义如下：

```mysql
drop table if exists t_sequence;
create table t_sequence
(
  seq_name varchar(64) not null primary key comment '序列名称',
  start_value bigint(20) not null default 1 comment '起始值',
  increment int(11) not null default 1 comment '增量值',
  updated_at timestamp not null default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP comment '更新时间'
) engine=InnoDB;
alter table t_sequence comment '序列表';
```

初始表中数据显示如下：
```mysql
mysql> insert into t_sequence(seq_name, start_value, increment, updated_at) values ('a', 1, 1, '2021-04-27 00:00:00');
Query OK, 1 row affected (0.04 sec)

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |           1 |         1 | 2021-04-27 00:00:00 |
+----------+-------------+-----------+---------------------+
1 row in set (0.00 sec)
```

### INSERT ... ON DUPLICATE KEY UPDATE语法

该语法用于插入或更新，使用前提是数据表必须包含唯一索引或主键。当执行的sql包含on duplicate key update子句时，插入的记录已经存在时，执行更新操作。 

```mysql
mysql> insert into t_sequence (seq_name, start_value, increment, updated_at) values ('a', 100, 1, now()) on duplicate key update start_value=200, increment=1, updated_at=now();
Query OK, 2 rows affected (0.04 sec)

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
+----------+-------------+-----------+---------------------+
1 row in set (0.00 sec)

mysql> insert into t_sequence (seq_name, start_value, increment, updated_at) values ('b', 1, 1, now()) on duplicate key update start_value=1, increment=1, updated_at=now();
Query OK, 1 row affected (0.05 sec)

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |           1 |         1 | 2021-04-27 17:13:31 |
+----------+-------------+-----------+---------------------+
2 rows in set (0.00 sec)
```

### REPLACE INTO语法

replace into语法的工作方式和insert完全相同，不同之处在于，如果表中旧行的主键或唯一索引与新行值相同，则在插入新行之前删除旧行。简述：要么插入，要么删除后再插入。

```mysql
mysql> replace into t_sequence (seq_name, start_value, increment, updated_at) values ('b', 10, 1, '2021-04-27 00:00:00');
Query OK, 2 rows affected (0.03 sec)

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |          10 |         1 | 2021-04-27 00:00:00 |
+----------+-------------+-----------+---------------------+
2 rows in set (0.00 sec)

mysql> replace into t_sequence(seq_name, start_value, increment, updated_at) values ('c', 1, 1, '2021-04-27 00:00:00');
Query OK, 1 row affected (0.03 sec)

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |          10 |         1 | 2021-04-27 00:00:00 |
| c        |           1 |         1 | 2021-04-27 00:00:00 |
+----------+-------------+-----------+---------------------+
3 rows in set (0.00 sec)
```

###  先插入后更新

先插入后更新主要运用MySql提供的insert ignore语法执行数据插入，执行update语句更新数据。

```mysql
mysql> insert ignore into t_sequence (seq_name, start_value, increment, updated_
at) values ('d', 1, 1, '2021-04-27 00:00:00');
Query OK, 1 row affected (0.05 sec)

mysql> update t_sequence
    -> set start_value= 1, increment=1, updated_at=now()
    -> where seq_name = 'd';
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |          10 |         1 | 2021-04-27 00:00:00 |
| c        |           1 |         1 | 2021-04-27 00:00:00 |
| d        |           1 |         1 | 2021-04-27 17:15:17 |
+----------+-------------+-----------+---------------------+
4 rows in set (0.00 sec)

mysql> insert ignore into t_sequence (seq_name, start_value, increment, updated_at) values ('d', 1, 1, '2021-04-27 00:00:00');
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> update t_sequence
    -> set start_value= 1, increment=1, updated_at='2021-04-27 00:00:00'
    -> where seq_name = 'd';
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |          10 |         1 | 2021-04-27 00:00:00 |
| c        |           1 |         1 | 2021-04-27 00:00:00 |
| d        |           1 |         1 | 2021-04-27 00:00:00 |
+----------+-------------+-----------+---------------------+
4 rows in set (0.00 sec)
```

### 先更新后插入

先更新后插入和先插入后更新的原理相同，都用到了insert ignore语法，只是insert 语句和update语句调换了顺序。这里更高级的用法是：先执行update语句，使用row_count()函数获取影响行数，再使用insert ignore ... select语法过滤where条件后执行插入操作。

PS：当待更新的数据在表中存在，执行update语句后，数据未发送变化时，row_count()函数结果可能返回0.

```mysql
mysql> update t_sequence
    -> set start_value= 1, increment=1, updated_at=now()
    -> where seq_name = 'e';
Query OK, 0 rows affected (0.00 sec)
Rows matched: 0  Changed: 0  Warnings: 0

mysql> select ROW_COUNT() into @cnt;
Query OK, 1 row affected (0.00 sec)

mysql> insert ignore into t_sequence (seq_name, start_value, increment, updated_at)
    -> select * from (select 'e' as seq_name, 1 as start_value, 1 as increment,'2021-04-27 00:00:00' as updated_at) t
    -> where 1 = if(@cnt > 0, 0, 1);
Query OK, 1 row affected (0.08 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |          10 |         1 | 2021-04-27 00:00:00 |
| c        |           1 |         1 | 2021-04-27 00:00:00 |
| d        |           1 |         1 | 2021-04-27 00:00:00 |
| e        |           1 |         1 | 2021-04-27 00:00:00 |
+----------+-------------+-----------+---------------------+
5 rows in set (0.00 sec)

mysql> update t_sequence
    -> set start_value= 1000, increment=1, updated_at=now()
    -> where seq_name = 'e';
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select ROW_COUNT() into @cnt;
Query OK, 1 row affected (0.00 sec)

mysql> insert ignore into t_sequence (seq_name, start_value, increment, updated_at)
    -> select * from (select 'e' as seq_name, 1 as start_value, 1 as increment,'2021-04-27 00:00:00' as updated_at) t
    -> where 1 = if(@cnt > 0, 0, 1);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |          10 |         1 | 2021-04-27 00:00:00 |
| c        |           1 |         1 | 2021-04-27 00:00:00 |
| d        |           1 |         1 | 2021-04-27 00:00:00 |
| e        |        1000 |         1 | 2021-04-27 17:20:37 |
+----------+-------------+-----------+---------------------+
5 rows in set (0.00 sec)
```

### 先删除后插入

先删除后插入：先执行delete语句删除数据，然后执行insert语句插入数据。 

```mysql
mysql> delete from t_sequence where seq_name = 'e';
Query OK, 1 row affected (0.04 sec)

mysql> insert into t_sequence(seq_name, start_value, increment, updated_at) values ('e', 10, 1, '2021-04-27 00:00:00');
Query OK, 1 row affected (0.06 sec)

mysql> select * from t_sequence;
+----------+-------------+-----------+---------------------+
| seq_name | start_value | increment | updated_at          |
+----------+-------------+-----------+---------------------+
| a        |         200 |         1 | 2021-04-27 17:12:27 |
| b        |          10 |         1 | 2021-04-27 00:00:00 |
| c        |           1 |         1 | 2021-04-27 00:00:00 |
| d        |           1 |         1 | 2021-04-27 00:00:00 |
| e        |          10 |         1 | 2021-04-27 00:00:00 |
+----------+-------------+-----------+---------------------+
5 rows in set (0.00 sec)
```

###  总结

毫无疑问，insert ... on duplicate key update语法是最适合数据表插入或更新操作的最佳方式。

严格意义上说，replace into语法和先删除后插入都不符合数据插入或更新的标准，执行过程存在删除操作，因此不推荐。

先插入后更新、先更新后插入的核心在于insert ignore语法，当插入的数据与主键或唯一索引冲突时，数据插入会被丢弃，且不会抛出异常。当数据表不存在主键和唯一索引时，先插入后更新、先更新后插入可以作为备选方案。结合row_count()和if()函数在where子句中对插入或更新影响行数进行过滤，可以减少一次sql语句执行，进而提升处理效率。

| 插入或更新实现方式                     | 前提条件                 | 推荐指数 | 备注                    |
| -------------------------------------- | ------------------------ | -------- | -------- |
| INSERT ... ON DUPLICATE KEY UPDATE语法 | 数据表存在主键或唯一索引 | ★★★★★ | 推荐 |
| REPLACE INTO语法                       | 数据表存在主键或唯一索引 | ★★★ | 存在删除操作 |
| 先插入后更新 |                          | ★★★ | 结合ROW_COUNT()可更高效 |
| 先更新后插入 |                          | ★★★ | 结合ROW_COUNT()可更高效 |
| 先删除后插入 |                          | ★★ | 存在删除操作 |




------

参考信息：

https://dev.mysql.com/doc/refman/5.7/en/insert.html

https://dev.mysql.com/doc/refman/5.7/en/insert-select.html

https://dev.mysql.com/doc/refman/5.7/en/insert-on-duplicate.html

https://dev.mysql.com/doc/refman/5.7/en/replace.html

https://dev.mysql.com/doc/refman/5.7/en/information-functions.html#function_row-count

https://docs.spring.io/spring-data/jpa/docs/2.5.0/reference/html/#jpa.entity-persistence.saving-entites