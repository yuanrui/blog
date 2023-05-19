---
title: 一个多表联合查询需求
date: 2020-02-13 22:10:00
tags:  
  - Sql  
categories:
  - Sql
toc: false
---
朋友做一个项目，遇到一个多表联合查询的需求。

- A表SYSTEM_ID和B表SYSTEM_ID关联；
- C表中ROLE_LIST字段，存储多个B表中的ROLE_ID值；

需要一个sql，当A表中SYSTEM_ID值为123时，找到B表和C表的关联，当B表满足SYSTEM_ID值为123时，包含其中的ROLE_ID数据，显示C表中NAME数据。例如查询结果为：james、lucy.

T_TABLE_A表

| ID   | SYSTEM_ID |
| ---- | --------- |
| 1    | `123`     |
| 2    | 234       |

T_TABLE_B表

| ID   | SYSTEM_ID | ROLE_ID |
| ---- | --------- | ------- |
| 1    | 222       | 667     |
| 2    | `123`     | `555`   |
| 3    | `123`     | `777`   |
| 4    | 234       | 567     |
| 5    | 234       | 231     |

T_TABLE_C表

| ID   | NAME    | ROLE_LIST     |
| ---- | ------- | ------------- |
| 1    | `james` | 667,`777`     |
| 2    | `lucy`  | 223,`555`,823 |
| 3    | tom     | 253,231       |
| 4    | max     | 123,712       |
| 5    | min     | 123,567       |

最终提供的sql如下：

```sql
select name from t_table_c c
where exists (
	select 1 from t_table_a a
	inner join t_table_b b on a.system_id = b.system_id
	where a.system_id = 123
    and c.role_list like '%' || b.role_id || '%' 
)
```

上面的查询sql采用的exists子句的方式，采用连接的方式也能完成相同的功能，具体实现见文末附录Sql.

从给的脱敏数据可以推测出各个表的功能。

- A表是权限（资源）表；
- B表是角色权限（资源）表；
- C表用户表；

一般使用单独的表来存储用户和角色的关联信息，这里很巧妙的采用以逗号分隔的方式存储多个角色编号。采用这种方式可以减少表连接，但处理存储和查询的复杂度更高。

单纯从数据上来看，由于采用的模糊查询，有可能出现错误的查询结果。

假设C表中存在以下数据。

| ID   | NAME | ROLE_LIST      |
| ---- | ---- | -------------- |
| 6    | kim  | `1777`,2211    |
| 7    | kenv | 220,`1555`,800 |

重新执行sql，你会发现查询结果里面包含：kim和kenv. 造成这个结果的原因是，模糊查询可以匹配ROLE_LIST列中部分数据，1777和1555分别包含777和555. 

通过这个简单示例，可以发现存储多个值时确实包含一些局限性。我们既想一列存储多个数值，又想消除歧义，那么怎么解决这个问题呢？

给存储的值添加一个前缀，也许是一个好办法，比如：r555,r777,r1777,r1555，前缀采用代表特定含义的单词或字母，加上实际的数值，可以构造出一个特殊的字符串。条件子语句：like '%r555'. 可以避免示例数据中的错误查询结果问题。

等等，好像还是有点问题。如果字符串为：r55,r555,r1555，何解。

当然如果存在这种情况，我们还有一个终极解决办法（适用于任何情况）：前缀+值+后缀

```
前缀+值+后缀

列存储示例：[55],[555],[1555]
条件子语句：like '%[55]%'，这里前缀='['、值='55'、后缀=']'
```

一般情况下，使用特定单词或字母作为特殊前缀加取值可以满级绝大多数情况，这种方式可读信更高。而需要严格意义上的避免误查询，可以采用：前缀+值+后缀。

```plsql
--附录Sql

create table t_table_a
(
    id int primary key,
    system_id int
);

create table t_table_b
(
    id int primary key,
    system_id int,
    role_id int
);

create table t_table_c
(
    id int primary key,
    name varchar2(20),
    role_list varchar2(200)
);

insert into t_table_a values(1, 123);
insert into t_table_a values(2, 234);
insert into t_table_a values(3, 100);

insert into t_table_b values(1, 222, 667);
insert into t_table_b values(2, 123, 555);
insert into t_table_b values(3, 123, 777);
insert into t_table_b values(4, 234, 567);
insert into t_table_b values(5, 234, 231);

insert into t_table_c values(1, 'james', '667,777');
insert into t_table_c values(2, 'lucy', '223,555,823');
insert into t_table_c values(3, 'tom', '253,231');
insert into t_table_c values(4, 'max', '123,712');
insert into t_table_c values(5, 'min', '123,567');
insert into t_table_c values(6, 'kim', '1777,2211');
insert into t_table_c values(7, 'kenv', '220,1555,800');

select name from t_table_c c
where exists (
	select 1 from t_table_a a
	inner join t_table_b b on a.system_id = b.system_id
	where a.system_id = 123
    and c.role_list like '%' || b.role_id || '%' 
);

select c.name, b.id from t_table_a a
inner join t_table_b b on a.system_id = b.system_id
inner join t_table_c c on c.role_list like '%' || b.role_id || '%' 
where a.system_id = 123;
```

On more thing. 

这sql在Oracle中跑起来，没有任何问题。但是朋友拿去改造后在mysql中运行，出现的查询结果依旧不正常。排查了很长一段时间，才找到原因：在mysql中 '%' || 'S001' || '%' 显示打印为：1. 正确的方式是使用concat函数。所以如果你在mysql中运行，需要这样构造sql.

```mysql
select c.name, b.id from t_table_a a
inner join t_table_b b on a.system_id = b.system_id
inner join t_table_c c on c.role_list like concat('%', b.role_id, '%') 
where a.system_id = 123;
```
