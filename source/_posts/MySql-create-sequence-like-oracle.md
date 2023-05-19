---
title: MySql实现生成序列值
date: 2021-04-01 11:32:00
tags:  
  - Sql
  - MySql 
categories:
  - Sql
  - MySql
toc: false
---
在Oracle中我们使用sequence来生成整型自增ID，而MySql不支持创建sequence. 虽然MySql中可以对主键设置AUTO_INCREMENT属性来实现表ID自增，但某些场景下需要提前获取ID标识领域对象，以实现业务生存周期的后续处理。在不支持sequence的其他RMDB中，一般使用序列表来实现对数列值的管理，并定义函数或存储过程用于获取更新序列值。

序列表的定义大同小异，一般包含序列名称(业务编号)、起始值(当前值)、增量(步长)等字段。

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

在MySql的分支项目MariaDB，从10.3开始支持创建sequence，可以使用Oracle的PL/SQL语法调用序列。MariaDB为了兼容PostgreSQL语法风格，定义了函数：setval()、nextval()、lastval()，分布用于设置当前序列值、获取下一个序列值、获取当前序列值。在PostgreSQL还定义了currval函数，用于返回序列最近获取的值，功能上和lastval相同；不同的是lastval不需要传递序列参数，获取的是当前会话的的序列的最近值。MariaDB中的lastval函数其实应该对应PostgreSQL中的currval，等同于Oracle中的sequence_name.currval.

为了和MariaDB保持兼容，我们定义名称相同的函数以实现序列功能。

```mysql
-- 启用函数创建功能
-- set global log_bin_trust_function_creators=1;

drop function if exists setval;
create function setval(vseq varchar(64), vstart bigint(20))
returns bigint(20)
begin
  insert into t_sequence(seq_name, start_value, increment, updated_at)
  values (vseq, vstart + 1, 1, CURRENT_TIMESTAMP())
  on duplicate key update start_value = vstart + increment;
  
  return vstart;
end;

drop function if exists nextval;
create function nextval(vseq varchar(64))
returns bigint(20)
begin
  set @next = null;
  update t_sequence
  set start_value = (@next := start_value) + increment
  where seq_name = vseq;
  
  return ifnull(@next, setval(vseq, 1));
end;

drop function if exists lastval;
create function lastval(vseq varchar(64))
returns bigint(20)
begin
  set @cur_val = null;
  select start_value into @cur_val 
  from t_sequence
  where seq_name = vseq;
  
  return ifnull(@cur_val, setval(vseq, 1));
end;
```

虽然每一个函数都有自动创建序列的功能，但是推荐在使用nextval函数前，初始化序列，避免高并发时获取到重复的初始值。使用示例如下。

```mysql
select setval('abc',1);  -- 返回1
select nextval('abc');   -- 返回2
select lastval('abc');   -- 返回3
```

序列的默认增量为1，业务需要高并发生成ID时，可适当扩充增量值，并通过编程实现批量获取ID值。以下是伪代码示例。

```
-- 执行sql:
insert into t_sequence (seq_name, start_value, increment) values ('system', 1, 100); -- 序列名称：system，初始值:1, 增量:100
批量获取序列值:
var startId = nextval('system'); // 返回1
var increment = 100;
var ids = new List<long>();
for(var i = 0; i < increment; i++) {
	ids.add(startId++);
}
```

本文所列举的是MySql的示例，此方案同样可用于其他不支持sequence的RMDB，比如Sql Server 2008、Sqlite...  

sql附件： {% asset_link mysql_sequence.sql %}

参考信息：

https://mariadb.com/kb/en/sequence-overview/

https://mariadb.com/kb/en/create-sequence/

https://mariadb.com/kb/en/sequence-functions/

https://www.postgresql.org/docs/current/sql-createsequence.html

https://www.postgresql.org/docs/current/functions-sequence.html