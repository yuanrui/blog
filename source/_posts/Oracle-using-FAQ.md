---
title: Oracle使用过程中的常见问题
date: 2017-06-28 15:00:00
tags:  
  - Oracle  
  - 开发笔记
categories:
  - Oracle
toc: false
---

Q:在命令行中执行存储过程后，存储过程中调用的dbms_output.put_line()方法中的内容未显示
A:需要执行命令: set serverout on;

Q:执行存储过程或函数的过程中报错:ORA-20000: ORU-10027: buffer overflow, limit of 2000 bytes
A:serverout的size默认为2000，需要修改size, 命令: set serveroutput on size 1000000;

Q:在Windows中登陆sqlplus时报:ORA-12560: TNS: 协议适配器错误
A:检查Oracle Net Manager中相关配置:监听程序和服务命名。测试服务命名的数据库连接是否可用。检查监听服务是否开启。
如果以上参数均正常，那么用sqlplus登陆的时候带上用户名、密码和服务名试试。eg:sqlplus scott/123456@local_instance ,这里的local_instance就是在Net Manager创建的服务命名。

Q:Oracle与表相关的视图DBA_,All_,USER_开头的视图含义
A:DBA_开头的视图显示当前数据库所有表的信息，ALL_开头的视图显示当前用户可以访问的所有视图，USER_开头的视图只显示属于当前用户的视图。

Q:在Windows安装Oracle Grid Infrastructure时报"PRCI-1113 : 目录 E:\app\11.2.0\grid 不存在"的错误提示。
A:打开目录C:\Program Files\Oracle\Inventory\ContentsXML删除inventory.xml文件。

Q:在Windows安装Oracle Grid Infrastructure时报"[INS-20802] 网格基础结构配置 失败。" 原因 - 插件的执行方法失败
A:查看Grid的安装目录下的\cfgtoollogs\crsconfig，找到目录中节点文件安装日志。查看安装日志中内容。安装过程中发现如下日志：
```
2017-09-25 09:56:34: ### Printing of configuration values complete ###
2017-09-25 09:56:34: HKLM/Software/Oracle/ocr/ is NOT configured

2017-09-25 09:56:34: HASH(0x4366ef8) registry key does not exist
2017-09-25 09:56:34: HASH(0x4366ef8) registry key does not exist
2017-09-25 09:56:34: Improper Oracle Clusterware configuration found on this host
2017-09-25 09:56:34: Deconfigure the existing cluster configuration before starting
2017-09-25 09:56:34: to configure a new Clusterware
2017-09-25 09:56:34: run 'E:\app\11.2.0\grid/crs/install/rootcrs.pl -deconfig' 
2017-09-25 09:56:34: to deconfigure existing failed configuration and then rerun root.bat
```
/*发现在注册表的HKLM/Software/Oracle路径中无ocr项。添加ocr项，同时在ocr中添加字符串值local_only=FALSE和ocrconfig_loc=+DATA
原因：再次安装时，提示[INS-40417] 安装程序检测到系统中有未使用的 Oracle 集群注册表 (OCR) 位置注册表键 (HKEY_LOCAL_MACHINE\Software\Oracle\Ocr)。手动删除了Ocr项。*/
update:这种情况，还是重装系统吧。

Q:Oracle RAC连接字符串设置
A:参考链接:http://www.oracle.com/technetwork/cn/articles/database-performance/oracle-rac-connection-mgmt-1650424-zhs.html

Q:Oracle RAC节点启动提示：CRS-1013:ASM 磁盘组中的 OCR 位置不可访问。
A:解决过程：日志中提示ASM磁盘不可访问，首先查看防火墙是否启用，如果启用则关闭。检查LISTENER是否正常。
最后使用命令启动：crsctl start has
