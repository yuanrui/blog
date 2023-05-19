---
title: 探讨下C#使用OPD.NET参数绑定问题
date: 2018-05-24 14:00:00
tags:  
  - Oracle  
  - 开发笔记
categories:
  - .Net
  - Ado.Net
toc: false
---
Oracle官方出的OPD.NET托管驱动对于ADO.NET的参数绑定方面，让人看不懂。
如下面这个代码，注意SAL、COMM、DEPTNO的值。
```
	const string cmdText = @"
insert into emp (EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) 
values (:EMPNO, :ENAME, :JOB, :MGR, :HIREDATE, :SAL, :COMM, :DEPTNO)";

	var connStr = "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.170)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=ebos)));User ID=scott;Password=tiger;";
	using (var conn = new OracleConnection(connStr))
	{
		using (var cmd = conn.CreateCommand())
		{
			cmd.CommandText = cmdText;
			cmd.Parameters.Add("EMPNO", OracleDbType.Int16, 1, ParameterDirection.Input);
			cmd.Parameters.Add("ENAME", OracleDbType.Varchar2, "ENAME".ToArray(), ParameterDirection.Input);
			cmd.Parameters.Add("JOB", OracleDbType.Varchar2, "JOB", ParameterDirection.Input);
			cmd.Parameters.Add("MGR", OracleDbType.Int32, 1, ParameterDirection.Input);
			cmd.Parameters.Add("HIREDATE", OracleDbType.Date, DateTime.Now, ParameterDirection.Input);
			cmd.Parameters.Add("DEPTNO", OracleDbType.Int32, 30, ParameterDirection.Input);
			cmd.Parameters.Add("COMM", OracleDbType.Decimal, 20, ParameterDirection.Input);
			cmd.Parameters.Add("SAL", OracleDbType.Decimal, 10, ParameterDirection.Input);

			conn.Open();
			var result = cmd.ExecuteNonQuery() > 0;
		}
	}
```
你想要的结果：SAL=10、COMM=20、DEPTNO=30。但实际上结果是这样的：
![默认情况下BindName为false](https://raw.githubusercontent.com/yuanrui/blog/master/_images/2018-05-24/OPD-NET-BindName-false.png)
很奇怪是吧，OPD.NET中默认情况下，参数绑定的不是依据名称而是根据顺序来传递的。
我们设置OracleCommand属性BindByName为true，再次执行插入语句代码。
```
cmd.BindByName = true;
```
EMPNO=2的行是我们想要的结果。
![设置OracleCommand属性BindByName为true](https://raw.githubusercontent.com/yuanrui/blog/master/_images/2018-05-24/OPD-NET-BindName-true-by-code.png)
那么每次通过设置BindByName=true，对于开发人员而言是一件痛苦的事情。当前不设置也没有关系，前提是你必须保证参数顺序的正确性。
有没有配置或全局属性可以将BindByName默认设置为true呢，Oracle官方考虑到这种情况，提供在web.config或app.config文件中设置默认值。
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <configSections>
        <section name="oracle.manageddataaccess.client"
          type="OracleInternal.Common.ODPMSectionHandler, Oracle.ManagedDataAccess, Version=4.122.1.0, Culture=neutral, PublicKeyToken=89b483f429c47342"/>
    </configSections>
    <system.data>
        <DbProviderFactories>
            <remove invariant="Oracle.ManagedDataAccess.Client"/>
            <add name="ODP.NET, Managed Driver" invariant="Oracle.ManagedDataAccess.Client" description="Oracle Data Provider for .NET, Managed Driver"
              type="Oracle.ManagedDataAccess.Client.OracleClientFactory, Oracle.ManagedDataAccess, Version=4.122.1.0, Culture=neutral, PublicKeyToken=89b483f429c47342"/>
        </DbProviderFactories>
    </system.data>
    <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
            <dependentAssembly>
                <publisherPolicy apply="no"/>
                <assemblyIdentity name="Oracle.ManagedDataAccess" publicKeyToken="89b483f429c47342" culture="neutral"/>
                <bindingRedirect oldVersion="4.121.0.0 - 4.65535.65535.65535" newVersion="4.122.1.0"/>
            </dependentAssembly>
        </assemblyBinding>
    </runtime>
    <oracle.manageddataaccess.client>
        <version number="*">
            <settings>
                <setting name="BindByName" value="true"/>
            </settings>
        </version>
    </oracle.manageddataaccess.client>
</configuration>
```
为了验证配置是否生效，注释代码cmd.BindByName=true, 再次运行代码，结果如下所示。
![在配置文件中设置BindByName为true](https://raw.githubusercontent.com/yuanrui/blog/master/_images/2018-05-24/OPD-NET-BindName-true-by-config.png)
需要说明的是，代码中设置BindByName的优先级高于配置。

参考链接：
https://docs.oracle.com/cd/E63277_01/win.121/e63268/InstallManagedConfig.htm#ODPNT8160
https://docs.oracle.com/en/database/oracle/oracle-database/12.2/odpnt/CommandBindByName.html#GUID-609B7F20-2444-4CBF-AC8A-A19907A626C8