---
title: Oracle学习笔记
date: 2016-10-25 15:37:33
tags:
  - Oracle
  - Sql
  - 开发笔记
  - 读书笔记
categories:
  - Oracle
  - Sql
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