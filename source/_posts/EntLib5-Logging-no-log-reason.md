---
title: EntLib5日志组件无法记录日志的可能原因
date: 2016-06-22 11:45:06
tags:
  - 开发笔记
categories:
  - .Net
toc: false
---
使用EntLib的Log组件记录日志时，经常有这样的需求，需要将日志记录到数据库中方便查询。但实际使用时经常会遇到网络问题或数据库端异常等原因，导致无法准确定位问题所在。出现这种情况时，需要将日志记录到本地文件中进行排查。
EntLib中的Log组件，使用监听器（Listener）的方式对日志进行分类记录。使用Listener时有顺序要求，假设有两个Listener，Database Trace Listener用于将日志记录到数据库中，Rolling Flat File Trace Listener将日志写入到文件中。
如果前一个Database Trace Listener不起作用或者执行过程中出现异常，后一个的Rolling Flat File Trace Listener也将不会执行。

假设你使用了自定义分类，还有一种可能原因是没有配置categorySources的子节点。
eg. 在记录日志时指定的分类为Program, 则必须在配置节点中添加如下配置:
```
<categorySources>
	<add switchValue="All" name="Program">
		<listeners>
			<add name="Formatted EventLog TraceListener"/>
			<add name="Rolling Flat File Trace Listener"/>
		</listeners>
	</add>
</categorySources>
```

LogEntry类用于封装传递需要记录的日志信息。LogEntry包含一个类型为IDictionary<string, object>的属性ExtendedProperties.使用时需要注意的是，字典的类型Value类型为object, 但建议往其中写入值时最好保存为string类型。因为每个Lintener都需要指定一个格式化器（Formatter），可以Formatter中指定显示内容，在显示ExtendedProperties中的Value值时可能不支持对应的object转换为string.

20170505 Update
在项目中引入Database Trace Listener时，也有可能造成无法记录日志
<add name="Database Trace Listener" type="Microsoft.Practices.EnterpriseLibrary.Logging.Database.FormattedDatabaseTraceListener, Microsoft.Practices.EnterpriseLibrary.Logging.Database" listenerDataType="Microsoft.Practices.EnterpriseLibrary.Logging.Database.Configuration.FormattedDatabaseTraceListenerData, Microsoft.Practices.EnterpriseLibrary.Logging.Database" databaseInstanceName="DefaultConnectionString" writeLogStoredProcName="Write_Log" addCategoryStoredProcName="Add_Log_Category" formatter="Json Log Formatter" traceOutputOptions="LogicalOperationStack, DateTime, Timestamp, ProcessId, ThreadId, Callstack"/>
可能原因是：Microsoft.Practices.EnterpriseLibrary.Logging.Database.dll没拷贝到对应的bin或程序根目录中，导致引用查找Microsoft.Practices.EnterpriseLibrary.Logging.Database.Configuration.FormattedDatabaseTraceListenerData时抛出异常: A configuration error has occurred. 

20170928 Update
在使用Database Trace Listener时内部调用的存储过程出错也会造成无日志的bug. 这个bug有点难排查，如果存储过程调用失败，Windows事件记录中也没有相关记录。
排查记录过程：发现日志表T_SYS_LOG_INFO多了一个字段，而存储过程Write_Log中内容是这样的：
```
--Oracle存储过程
create or replace procedure Write_Log(
  EventID INTEGER,
  Priority INTEGER,
  Severity NVARCHAR2,
  Title NVARCHAR2,
  Timestamp DATE,
  MachineName NVARCHAR2,
  AppDomainName NVARCHAR2,
  ProcessID NVARCHAR2,
  ProcessName NVARCHAR2,
  ThreadName NVARCHAR2,
  Win32ThreadId NVARCHAR2,
  Message NVARCHAR2,
  FormattedMessage NVARCHAR2,
  LogId out INTEGER) is
  v_ID INTEGER;
begin
  SELECT SEQ_T_SYS_LOG_INFO.NEXTVAL INTO v_ID FROM DUAL;
  INSERT INTO T_SYS_LOG_INFO
  VALUES(v_ID, EventID, Priority, Severity, Title, Timestamp, MachineName, AppDomainName, ProcessID, ProcessName, ThreadName, Win32ThreadId, Message, FormattedMessage);
  SELECT v_ID INTO LogId FROM DUAL;
end Write_Log;
```
添加的列中无默认值，导致在调用这个存储过程时，存储过程中的sql执行失败。修改存储过程INSERT语句，指定列。
```
--Oracle存储过程
create or replace procedure Write_Log(
  EventID INTEGER,
  Priority INTEGER,
  Severity NVARCHAR2,
  Title NVARCHAR2,
  Timestamp DATE,
  MachineName NVARCHAR2,
  AppDomainName NVARCHAR2,
  ProcessID NVARCHAR2,
  ProcessName NVARCHAR2,
  ThreadName NVARCHAR2,
  Win32ThreadId NVARCHAR2,
  Message NVARCHAR2,
  FormattedMessage NVARCHAR2,
  LogId out INTEGER) is
  v_ID INTEGER;
begin
  SELECT SEQ_T_SYS_LOG_INFO.NEXTVAL INTO v_ID FROM DUAL;
  INSERT INTO T_SYS_LOG_INFO
  (ID, EVENT_ID, PRIORITY, SEVERITY, TITLE, TIMESTAMP, MACHINE_NAME, APPDOMAIN_NAME, PROCESS_ID, PROCESS_NAME, THREAD_NAME, WIN32_THREAD_ID, MESSAGE, FORMATTED_MESSAGE)
  VALUES(v_ID, EventID, Priority, Severity, Title, Timestamp, MachineName, AppDomainName, ProcessID, ProcessName, ThreadName, Win32ThreadId, Message, FormattedMessage);
  SELECT v_ID INTO LogId FROM DUAL;
end Write_Log;
```
