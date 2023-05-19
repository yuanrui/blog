---
title: Oracle中的导入导出
date: 2016-06-16 09:28:34
tags:
  - Oracle
  - Sql
  - 开发笔记
categories:
  - Oracle
---
需要将Oracle中的数据进行导入导出时，可以使用imp和exp命令进行操作。exp命令可以将数据库服务器端的数据导出到本地的.dmp文件中，而imp命令可以将.dmp文件中的数据还原到数据库中。
如果不了解详细的命令参数，可以使用exp help=y，imp help=y进行查询。
#### 数据导出导入示例
数据导出.bat
```
exp 用户名/密码 file='文件绝对路径.dmp' filesize=10G tables=数据表1,数据表2,数据表3 grants=n indexes=n triggers=n record=n constraints=n rows=y feedback=10000
pause;
```
文件导入.bat
```
imp 用户名/密码 file='文件绝对路径.dmp' tables=数据表1,数据表2,数据表3 DATA_ONLY=y
pause;
```
#### exp相关参数
```
C:\Users\Administrator>exp help=y

Export: Release 11.2.0.1.0 - Production on 星期四 6月 16 10:02:16 2016

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.



通过输入 EXP 命令和您的用户名/口令, 导出
操作将提示您输入参数:

     例如: EXP SCOTT/TIGER

或者, 您也可以通过输入跟有各种参数的 EXP 命令来控制导出
的运行方式。要指定参数, 您可以使用关键字:

     格式:  EXP KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
     例如: EXP SCOTT/TIGER GRANTS=Y TABLES=(EMP,DEPT,MGR)
               或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表

USERID 必须是命令行中的第一个参数。

关键字   说明 (默认值)         关键字      说明 (默认值)
--------------------------------------------------------------------------
USERID   用户名/口令           FULL        导出整个文件 (N)
BUFFER   数据缓冲区大小        OWNER        所有者用户名列表
FILE     输出文件 (EXPDAT.DMP)  TABLES     表名列表
COMPRESS  导入到一个区 (Y)   RECORDLENGTH   IO 记录的长度
GRANTS    导出权限 (Y)          INCTYPE     增量导出类型
INDEXES   导出索引 (Y)         RECORD       跟踪增量导出 (Y)
DIRECT    直接路径 (N)         TRIGGERS     导出触发器 (Y)
LOG      屏幕输出的日志文件    STATISTICS    分析对象 (ESTIMATE)
ROWS      导出数据行 (Y)        PARFILE      参数文件名
CONSISTENT 交叉表的一致性 (N)   CONSTRAINTS  导出的约束条件 (Y)

OBJECT_CONSISTENT    只在对象导出期间设置为只读的事务处理 (N)
FEEDBACK             每 x 行显示进度 (0)
FILESIZE             每个转储文件的最大大小
FLASHBACK_SCN        用于将会话快照设置回以前状态的 SCN
FLASHBACK_TIME       用于获取最接近指定时间的 SCN 的时间
QUERY                用于导出表的子集的 select 子句
RESUMABLE            遇到与空格相关的错误时挂起 (N)
RESUMABLE_NAME       用于标识可恢复语句的文本字符串
RESUMABLE_TIMEOUT    RESUMABLE 的等待时间
TTS_FULL_CHECK       对 TTS 执行完整或部分相关性检查
TABLESPACES          要导出的表空间列表
TRANSPORT_TABLESPACE 导出可传输的表空间元数据 (N)
TEMPLATE             调用 iAS 模式导出的模板名

成功终止导出, 没有出现警告。
```
#### imp相关参数
```
C:\Users\Administrator>imp help=y

Import: Release 11.2.0.1.0 - Production on 星期四 6月 16 10:22:54 2016

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.



通过输入 IMP 命令和您的用户名/口令, 导入
操作将提示您输入参数:

     例如: IMP SCOTT/TIGER

或者, 可以通过输入 IMP 命令和各种参数来控制导入
的运行方式。要指定参数, 您可以使用关键字:

     格式:  IMP KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
     例如: IMP SCOTT/TIGER IGNORE=Y TABLES=(EMP,DEPT) FULL=N
               或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表

USERID 必须是命令行中的第一个参数。

关键字   说明 (默认值)        关键字      说明 (默认值)
--------------------------------------------------------------------------
USERID   用户名/口令           FULL       导入整个文件 (N)
BUFFER   数据缓冲区大小        FROMUSER    所有者用户名列表
FILE     输入文件 (EXPDAT.DMP)  TOUSER     用户名列表
SHOW     只列出文件内容 (N)     TABLES      表名列表
IGNORE   忽略创建错误 (N)    RECORDLENGTH  IO 记录的长度
GRANTS   导入权限 (Y)          INCTYPE     增量导入类型
INDEXES   导入索引 (Y)         COMMIT       提交数组插入 (N)
ROWS     导入数据行 (Y)        PARFILE      参数文件名
LOG     屏幕输出的日志文件    CONSTRAINTS    导入限制 (Y)
DESTROY                覆盖表空间数据文件 (N)
INDEXFILE              将表/索引信息写入指定的文件
SKIP_UNUSABLE_INDEXES  跳过不可用索引的维护 (N)
FEEDBACK               每 x 行显示进度 (0)
TOID_NOVALIDATE        跳过指定类型 ID 的验证
FILESIZE               每个转储文件的最大大小
STATISTICS             始终导入预计算的统计信息
RESUMABLE              在遇到有关空间的错误时挂起 (N)
RESUMABLE_NAME         用来标识可恢复语句的文本字符串
RESUMABLE_TIMEOUT      RESUMABLE 的等待时间
COMPILE                编译过程, 程序包和函数 (Y)
STREAMS_CONFIGURATION  导入流的一般元数据 (Y)
STREAMS_INSTANTIATION  导入流实例化元数据 (N)
DATA_ONLY              仅导入数据 (N)

下列关键字仅用于可传输的表空间
TRANSPORT_TABLESPACE 导入可传输的表空间元数据 (N)
TABLESPACES 将要传输到数据库的表空间
DATAFILES 将要传输到数据库的数据文件
TTS_OWNERS 拥有可传输表空间集中数据的用户

成功终止导入, 没有出现警告。
```
