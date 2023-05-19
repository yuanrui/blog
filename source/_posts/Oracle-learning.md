---
title: Oracle学习笔记
date: 2016-11-25 10:17:55
tags:
  - Oracle
  - Sql
  - 开发笔记
  - 读书笔记
categories:
  - Sql
  - Oracle
---
### 用户相关
创建用户
```
--用户名test 密码123456
create user test identified by 123456;
```
修改用户密码
```
--用户名test 密码test
alter user test identified by test;
```
为用户指定默认表空间
```
--默认表空间test_DB 临时表空间test_temp
alter user test
default tablespace test_DB
temporary tablespace test_temp;
```
授予用户权限
```
grant 
　　CREATE SESSION, CREATE ANY TABLE, CREATE ANY VIEW ,CREATE ANY INDEX, CREATE ANY PROCEDURE,
　　ALTER ANY TABLE, ALTER ANY PROCEDURE,
　　DROP ANY TABLE, DROP ANY VIEW, DROP ANY INDEX, DROP ANY PROCEDURE,
　　SELECT ANY TABLE, INSERT ANY TABLE, UPDATE ANY TABLE, DELETE ANY TABLE
　　to test;
grant resource, dba to test;
```
回收权限
```
revoke resource, dba from test;
```
### 表空间管理
表空间大小与数据块大小(db_block_size)有关

| 数据块大小 | 表空间文件最大值M | 表空间文件最大值G | 
|:------------- |:------------- |
| 4k | 16383M | 16G |
| 8k | 32767M | 32G |
| 16k | 65535M | 64G |
| 32k | 131072M | 128G |
| 64k | 262144M | 256G |

数据块是4k的数据库，单个表空间数据文件的限制是小于16G，而不是小于等于16G。

查询数据块大小大小
```
show parameter db_block_size
```

查询表空间使用情况
```
SELECT UPPER(F.TABLESPACE_NAME) AS "表空间名称",
       ROUND(D.AVAILB_BYTES,2) AS "表空间大小(G)",
       ROUND(D.MAX_BYTES,2) AS "最终表空间大小(G)",
       ROUND((D.AVAILB_BYTES - F.USED_BYTES),2) AS "已使用空间(G)",
       TO_CHAR(ROUND((D.AVAILB_BYTES - F.USED_BYTES) / D.AVAILB_BYTES * 100, 2), '999.99') AS "使用比",
       ROUND(F.USED_BYTES, 6) AS "空闲空间(G)",
       F.MAX_BYTES AS "最大块(M)"
FROM
  ( SELECT TABLESPACE_NAME,
           ROUND(SUM(BYTES) / (1024 * 1024 * 1024), 6) USED_BYTES,
           ROUND(MAX(BYTES) / (1024 * 1024 * 1024), 6) MAX_BYTES
   FROM SYS.DBA_FREE_SPACE
   GROUP BY TABLESPACE_NAME) F,
  (SELECT DD.TABLESPACE_NAME,
          ROUND(SUM(DD.BYTES) / (1024 * 1024 * 1024), 6) AVAILB_BYTES,
          ROUND(SUM(DECODE(DD.MAXBYTES, 0, DD.BYTES, DD.MAXBYTES))/(1024*1024*1024),6) MAX_BYTES
   FROM SYS.DBA_DATA_FILES DD
   GROUP BY DD.TABLESPACE_NAME) D
WHERE D.TABLESPACE_NAME = F.TABLESPACE_NAME
ORDER BY 4 DESC
```

查询表空间文件地址、是否自动增长、数据文件最大值、自动增长值(8k block)
```
select TABLESPACE_NAME,FILE_NAME,AUTOEXTENSIBLE,MAXBYTES / 1024 / 1024 / 1024,INCREMENT_BY*8192/1024/1024 from dba_data_files
```
增加表空间数据文件
```
alter tablespace test_DB
add datafile 'D:\ORADB\test_DB_2.DBF' size 32M
AUTOEXTEND ON
NEXT 32M MAXSIZE 32767M;
```
调整数据文件大小
```
alter database datafile 'D:\ORADB\test_DB_2.DBF'
RESIZE 64M;
```
设置已存在数据文件为自动增长并设置最大值
```
alter database datafile 'D:\ORADB\test_DB_2.DBF'
AUTOEXTEND ON NEXT 32M MAXSIZE 32767M;
```
删除数据文件
```
alter tablespace test_DB
drop datafile 'D:\ORADB\test_DB_2.DBF';
```
移动表空间数据文件
```
--1.将表空间离线
alter tablespace test_DB offline normal;
--2.移动数据文件目录
--3.重命名表空间文件
alter database rename file 'D:\ORADB\test_DB.DBF' to 'D:\Oracle\Data\test_DB.DBF';
--4.表空间联机
alter tablespace test_DB online;
```

查看表空间或索引占用的存储空间大小
```
select segment_name, segment_type, sum(bytes)/1024/1024 MB
from user_segments  
group by segment_name, segment_type
order by 3 desc 
```

查询引用主键的外键以及外键表
```
select a.Owner 外键拥有者,
    a.Table_Name 外键表,
    c.Column_Name 外键列,
    b.Owner 主键拥有者,
    b.Table_Name 主键表,
    d.Column_Name 主键列,
    c.Constraint_Name 外键名,
    d.Constraint_Name 主键名
from User_Constraints a, 
    user_Constraints b, 
    user_Cons_Columns c, --外键表
    user_Cons_Columns d  --主键表
where a.r_Constraint_Name = b.Constraint_Name 
    and a.Constraint_Type = 'R' 
    and b.Constraint_Type = 'P' 
    and a.r_Owner = b.Owner 
    and a.Constraint_Name = c.Constraint_Name 
    and b.Constraint_Name = d.Constraint_Name 
    and a.Owner = c.Owner 
    and a.Table_Name = c.Table_Name 
    and b.Owner = d.Owner 
    and b.Table_Name = d.Table_Name
    and b.Table_Name = 'EMP'
```