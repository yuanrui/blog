---
title: Oracle Sql递归
date: 2017-04-28 17:55:00
tags:
  - Oracle
  - Sql
  - 开发笔记
  - 读书笔记
categories:
  - Sql
  - Oracle
toc: false
---

最近在做一个用户管理方面的需求，用户只能修改自己创建的用户以及派生用户，提交保存时需要对用户进行判断和识别是否为派生用户
```
--测试用脚本
create table T_SYS_USER
(
  id         INTEGER,
  name       VARCHAR2(20),
  password   VARCHAR2(20),
  created_by VARCHAR2(20)
);
/
--id为主键 name唯一
insert into T_SYS_USER values (1, 'admin', '', 'admin');
insert into T_SYS_USER values (2, 'root', '', 'admin');
insert into T_SYS_USER values (3, 'user', '', 'root');
insert into T_SYS_USER values (4, 'dba', '', 'user');
insert into T_SYS_USER values (5, 'yuanrui', '', 'dba');
insert into T_SYS_USER values (6, 'banana', '', 'yuanrui');
insert into T_SYS_USER values (7, 'dog', '', 'banana');
insert into T_SYS_USER values (8, 'cat', '', 'dog');
insert into T_SYS_USER values (9, 'pig', '', 'cat');
insert into T_SYS_USER values (10, 'fish', '', 'pig');

commit
/
```

查询用户yuanrui的子账户
```
select level, id, name, created_by 
from T_SYS_USER
start with id = 5
connect by prior UPPER(name) = UPPER(created_by)
```
结果

| LEVLE | ID | NAME | CREATED_BY |
|:------------- |:------------- |:------------- |
| 1	 | 5	 | yuanrui	 | dba |
| 2	 | 6	 | banana	 | yuanrui |
| 3	 | 7	 | dog	 | banana |
| 4	 | 8	 | cat	 | dog |
| 5	 | 9	 | pig	 | cat |
| 6	 | 10	 | fish	 | pig |

查询用户yuanrui的父账户
```
select  level, id, name, created_by 
from T_SYS_USER
start with id = 5
connect by prior UPPER(created_by) = UPPER(name)
```
结果

| LEVLE | ID | NAME | CREATED_BY |
|:------------- |:------------- |:------------- |
| 1	 | 5	 | yuanrui	 | dba |
| 2	 | 4	 | dba	 | user |
| 3	 | 3	 | user	 | root |
| 4	 | 2	 | root	 | admin |
| 5	 | 1	 | admin	 | 　 |

在实际使用情况中，递归查询有时候还是会抛出异常，主要原因是数据不正确造成的。
举个例子
```
update T_SYS_USER
set created_by = name
where id = 1;
commit
/
select  level, id, name, created_by 
from T_SYS_USER
start with id = 5
connect by prior UPPER(created_by) = UPPER(name)
```
执行时会抛出异常:ORA-01436:用户数据中的 CONNECT BY 循环
在CONNECT BY后加上nocycle可以结束循环调用
```
select  level, id, name, created_by 
from T_SYS_USER
start with id = 5
connect by prior UPPER(created_by) = UPPER(name)
```
注意结果有所变化，编号为1的数据行被排除掉了

| LEVLE | ID | NAME | CREATED_BY |
|:------------- |:------------- |:------------- |
| 1	 | 5	 | yuanrui	 | dba |
| 2	 | 4	 | dba	 | user |
| 3	 | 3	 | user	 | root |
| 4	 | 2	 | root	 | admin |

以前玩MS SQL的时候，玩过用公用表表达式(Common Table Expressions)做递归。网上查找Oracle Sql递归大多数结果都是关于用CONNECT BY来实现的，在Oracle中其实也可以用CTE来做递归。注意版本问题，本地测试环境Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production可用，听说11g r2之前的版本不支持，求证？
给一个CTE版本查询示例，查询结果是差不多的
```
--查询用户yuanrui的子账户
with userCte(level_, id, name, created_by) as
(
     select 1 as level_, id, name, created_by
     from T_SYS_USER
     where id =5
     union all
     select b.level_ + 1 as level_, a.id, a.name, a.created_by
     from T_SYS_USER a, userCte b
     where UPPER(a.created_by) = UPPER(b.name) 
     and a.created_by <> a.name
)
select * from userCte;

--查询用户yuanrui的父账户
with userCte(level_, id, name, created_by) as
(
     select 1 as level_, id, name, created_by
     from T_SYS_USER
     where id =5
     union all
     select b.level_ + 1 as level_, a.id, a.name, a.created_by
     from T_SYS_USER a, userCte b
     where UPPER(a.name) = UPPER(b.created_by) 
     and a.created_by <> a.name
)
select * from userCte;
```
where id=5为起始条件，union all 后面的子查询where UPPER(a.name) = UPPER(b.created_by) 为连接条件
注意where条件中的and a.created_by <> a.name, 这个条件主要用于结束循环(对应上文中的nocycle)

其他参考:http://www.cnblogs.com/yingsong/p/5035907.html
