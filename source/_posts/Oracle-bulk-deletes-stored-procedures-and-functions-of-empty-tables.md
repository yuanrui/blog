---
title: Oracle批量删除空表的存储过程和函数
date: 2023-05-12 11:00:00
tags:
  - Sql
  - Oracle
  - 开发笔记
categories:
  - Sql
  - Oracle
toc: true
---
### 前言

程序配置的创建分表语句有误，创建了大量的未使用的分表，需要执行sql循环删除空表。这里的空表指的数据表行数为空，保险起见数据表存在记录时不删除该表。

### 定义存储过程

```sql
-- 删除空表的存储过程
create or replace procedure SP_DROP_EMPTY_TABLE(owner varchar2, tableName varchar2)
is
  v_cnt number;
  v_index number := 0;
  v_owner varchar(100) := owner;
  v_table varchar2(100) := tableName;
  v_sql varchar2(200) := '';
  cursor cur_tables(c_owner varchar2, c_tableName varchar2) is select TABLE_NAME from all_tables where OWNER = UPPER(c_owner) and TABLE_NAME like UPPER(c_tableName) || '%' ;
begin
  dbms_output.enable(buffer_size => null);
  open cur_tables(owner, tableName); -- 打开游标
  loop
      fetch cur_tables into v_table; -- 提取游标
      exit when cur_tables%notfound;
      v_index := v_index + 1;
      v_sql := 'select count(*) from ' || v_table;
      execute immediate v_sql into v_cnt;
      if v_cnt = 0 then 
        v_sql := 'drop table '|| v_table;
        execute immediate v_sql;
        dbms_output.put_line('table:' || v_table || ' is empty, drop success.');
      else
        dbms_output.put_line('table:' || v_table || ' has records, drop fail. total count=' || v_cnt);
      end if;
  end loop;
  close cur_tables; -- 关闭游标

  if v_index = 0 then
    dbms_output.put_line('table:' || tableName || ' does not exist.');
  end if;
end;
/
```

### 执行存储过程

```sql
call SP_DROP_EMPTY_TABLE('test', 't_table_xxx');
```

### 定义函数

```sql
-- 删除空表的函数
create or replace function FUN_DROP_EMPTY_TABLE(owner  VARCHAR2, tableName  VARCHAR2)
return clob
as
  v_result clob := '';
  v_cnt number;  
  v_owner varchar2(100) := owner;
  v_table varchar2(100) := tableName;
  v_sql varchar2(200) := '';
  cursor cur_tables(c_owner varchar2, c_tableName varchar2) is select TABLE_NAME from all_tables where OWNER = UPPER(c_owner) and TABLE_NAME like UPPER(c_tableName) || '%' ;
begin
  open cur_tables(owner, tableName); -- 打开游标
  loop
      fetch cur_tables into v_table; -- 提取游标
      exit when cur_tables%notfound;
      v_sql := 'select count(*) from ' || v_table;
      execute immediate v_sql into v_cnt;
      if v_cnt = 0 then 
        v_sql := 'drop table '|| v_table;
        execute immediate v_sql;
        v_result := v_result || 'table:' || v_table || ' is empty, drop success.' || chr(13);
      else
        v_result := v_result || 'table:' || v_table || ' has records, drop fail. total count=' || v_cnt || chr(13);
      end if;
  end loop;
  close cur_tables; -- 关闭游标

  if v_result = '' or v_result is null then
    return 'table:' || tableName || ' does not exist';
  end if;

  return v_result;
end;
/
```

编写删除空表的函数版踩了多个坑。函数执行一般执行使用select FUN_DROP_EMPTY_TABLE('test', 't_table_xxx') from dual，由于存在DDL操作，不能使用select语句，否则会出现如下提示。

```
> ORA-14552: cannot perform a DDL, commit or rollback inside a query or DML 
ORA-06512: at "TEST.FUN_DROP_EMPTY_TABLE", line 19
```

一度以为函数不可用的时候，发现可以使用begin end包裹语句，在内部调用dbms_output.put_line显示执行结果。
但是又引出另外一个坑，Oracle 12c以前的版本，函数需要返回值。最开始编写这个函数时，把表的删除结果拼接后返回类型为varchar2，当数据表过多时函数执行后会抛出缓冲区过小异常。

```
> ORA-06502: PL/SQL: numeric or value error: character string buffer too small
ORA-06512: at "TEST.FUN_DROP_EMPTY_TABLE", line 20
ORA-06512: at line 3
```

要解决缓冲区问题要么减少返回值，要么将返回类型修改为clob. 函数返回字符串过长时，有可能导致dbms_output.put_line输出异常或不输出...
最后，最好使用存储过程。

### 执行函数

```sql
begin
-- dbms_output.enable(buffer_size => null);
dbms_output.put_line(FUN_DROP_EMPTY_TABLE('test', 't_table_xxx'));
end;
```

### 直接执行sql

考虑到部分账户可能没有执行存储过程和函数的权限，这里给出个直接执行sql的版本。

```sql
-- 直接执行语句
declare
  v_owner varchar(100) := 'admin'; -- 必须输入项: 账户, 即拥有者
  v_table varchar2(100) := 'T_XXX_YYMMDD'; -- 必须输入项: 表名或表名前缀
  v_cnt number;
  v_index number := 0;
  v_sql varchar2(200) := '';
  cursor cur_tables(c_owner varchar2, c_tableName varchar2) is select TABLE_NAME from all_tables where OWNER = UPPER(c_owner) and TABLE_NAME like UPPER(c_tableName) || '%' ;
begin
  dbms_output.enable(buffer_size => null);
  open cur_tables(owner, tableName); -- 打开游标
  loop
      fetch cur_tables into v_table; -- 提取游标
      exit when cur_tables%notfound;
      v_index := v_index + 1;
      v_sql := 'select count(*) from ' || v_table;
      execute immediate v_sql into v_cnt;
      if v_cnt = 0 then 
        v_sql := 'drop table '|| v_table;
        execute immediate v_sql;
        dbms_output.put_line('table:' || v_table || ' is empty, drop success.');
      else
        dbms_output.put_line('table:' || v_table || ' has records, drop fail. total count=' || v_cnt);
      end if;
  end loop;
  close cur_tables; -- 关闭游标

  if v_index = 0 then
    dbms_output.put_line('table:' || tableName || ' does not exist.');
  end if;
end;
```

附录：[MySQL批量删除空表的存储过程](https://github.com/yuanrui/blog/issues/49)