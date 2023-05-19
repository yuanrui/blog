---
title: Asp.Net Mvc 3升级到Mvc 4
date: 2017-05-08 17:30:00
tags:
  - Asp.Net
  - 开发笔记  
categories:
  - .Net
  - Asp.Net
toc: false
---
本文的前提是安装了Asp.Net Mvc 3.0 RTM, 同时安装Mvc 4.0
新建一个Empty Mvc3项目, 创建HomeController控制器，并创建Index视图
使用nuget安装如下package:
```
Install-Package Microsoft.AspNet.Razor -Version 2.0.30506
Install-Package Microsoft.AspNet.WebPages -Version 2.0.30506
Install-Package Microsoft.AspNet.Mvc -Version 4.0.40804
Install-Package Microsoft.AspNet.Mvc.zh-Hans -Version 4.0.40804
```
使用nuget完成安装dll后,需要修改根目录中的Web.config和Views文件夹中的Web.config
打开根目录中的Web.config,将System.Web.Helpers的版本修改为2.0.0.0, 将System.Web.Mvc的版本修改为4.0.0.1,将System.Web.WebPages的版本修改为2.0.0.0 
<system.web>中的最终配置如下
```
<compilation debug="true" targetFramework="4.0">
  <assemblies>
	<add assembly="System.Web.Abstractions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
	<add assembly="System.Web.Helpers, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
	<add assembly="System.Web.Routing, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
	<add assembly="System.Web.Mvc, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
	<add assembly="System.Web.WebPages, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
  </assemblies>
</compilation>
```
在<runtime>节点中添加如下配置
```
<assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
	<dependentAssembly>
		<assemblyIdentity name="System.Web.Helpers" publicKeyToken="31bf3856ad364e35"/>
		<bindingRedirect oldVersion="1.0.0.0" newVersion="2.0.0.0"/>
	</dependentAssembly>
	<dependentAssembly>
		<assemblyIdentity name="System.Web.WebPages" publicKeyToken="31bf3856ad364e35"/>
		<bindingRedirect oldVersion="1.0.0.0" newVersion="2.0.0.0"/>
	</dependentAssembly>
	<dependentAssembly>
		<assemblyIdentity name="System.Web.Mvc" publicKeyToken="31bf3856ad364e35"/>
		<bindingRedirect oldVersion="3.0.0.0-4.0.0.1" newVersion="4.0.0.1"/>
	</dependentAssembly>
</assemblyBinding>
```
打开Views文件夹中的Web.config,将原先的configSections节点项中Razor版本修改为2.0.0.0, 同时将这个文件中的Mvc版本修改为4.0.0.1 

根目录中的Web.config最终内容
```
<?xml version="1.0" encoding="utf-8"?>
<!--
  有关如何配置 ASP.NET 应用程序的详细信息，请访问
  http://go.microsoft.com/fwlink/?LinkId=152368
  -->

<configuration>
  <appSettings>
    <add key="ClientValidationEnabled" value="true"/> 
    <add key="UnobtrusiveJavaScriptEnabled" value="true"/> 
  </appSettings>
    
  <system.web>
    <compilation debug="true" targetFramework="4.0">
      <assemblies>
        <add assembly="System.Web.Abstractions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
        <add assembly="System.Web.Helpers, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
        <add assembly="System.Web.Routing, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
        <add assembly="System.Web.Mvc, Version=4.0.0.1, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
        <add assembly="System.Web.WebPages, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
      </assemblies>
    </compilation>

    <authentication mode="Forms">
      <forms loginUrl="~/Account/LogOn" timeout="2880"/>
    </authentication>

    <pages>
      <namespaces>
        <add namespace="System.Web.Helpers"/>
        <add namespace="System.Web.Mvc"/>
        <add namespace="System.Web.Mvc.Ajax"/>
        <add namespace="System.Web.Mvc.Html"/>
        <add namespace="System.Web.Routing"/>
        <add namespace="System.Web.WebPages"/>
      </namespaces>
    </pages>
  </system.web>

  <system.webServer>
    <validation validateIntegratedModeConfiguration="false"/>
    <modules runAllManagedModulesForAllRequests="true"/>
  </system.webServer>

  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="System.Web.Mvc" publicKeyToken="31bf3856ad364e35"/>
        <bindingRedirect oldVersion="3.0.0.0-4.0.0.1" newVersion="4.0.0.1"/>
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>
```

Views文件夹中的Web.config中的最终内容
```
<?xml version="1.0" encoding="utf-8"?>

<configuration>
  <configSections>
    <sectionGroup name="system.web.webPages.razor" type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
      <section name="host" type="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
      <section name="pages" type="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
    </sectionGroup>
  </configSections>

  <system.web.webPages.razor>
    <host factoryType="System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
    <pages pageBaseType="System.Web.Mvc.WebViewPage">
      <namespaces>
        <add namespace="System.Web.Mvc" />
        <add namespace="System.Web.Mvc.Ajax" />
        <add namespace="System.Web.Mvc.Html" />
        <add namespace="System.Web.Routing" />
      </namespaces>
    </pages>
  </system.web.webPages.razor>

  <appSettings>
    <add key="webpages:Enabled" value="false" />
  </appSettings>

  <system.web>
    <httpHandlers>
      <add path="*" verb="*" type="System.Web.HttpNotFoundHandler"/>
    </httpHandlers>

    <!--
        在视图页面中启用请求验证将导致验证在
        控制器已对输入进行处理后发生。默认情况下，
        MVC 在控制器处理输入前执行请求验证。
        若要更改此行为，请对控制器或操作
        应用 ValidateInputAttribute。
    -->
    <pages
        validateRequest="false"
        pageParserFilterType="System.Web.Mvc.ViewTypeParserFilter, System.Web.Mvc, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"
        pageBaseType="System.Web.Mvc.ViewPage, System.Web.Mvc, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"
        userControlBaseType="System.Web.Mvc.ViewUserControl, System.Web.Mvc, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
      <controls>
        <add assembly="System.Web.Mvc, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" namespace="System.Web.Mvc" tagPrefix="mvc" />
      </controls>
    </pages>
  </system.web>

  <system.webServer>
    <validation validateIntegratedModeConfiguration="false" />

    <handlers>
      <remove name="BlockViewHandler"/>
      <add name="BlockViewHandler" path="*" verb="*" preCondition="integratedMode" type="System.Web.HttpNotFoundHandler" />
    </handlers>
  </system.webServer>
</configuration>
```

重新编译后运行，自动跳转到/Home/Index

Mvc 4的最新版本为4.0.0.1, 在Nuget上可以下载, 为了方便还原packages.config如下
```
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Antlr" version="3.5.0.2" targetFramework="net40" />
  <package id="jQuery" version="1.12.4" targetFramework="net40" />
  <package id="Microsoft.AspNet.Mvc" version="4.0.40804.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.Mvc.zh-Hans" version="4.0.40804.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.Razor" version="2.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.Razor.zh-Hans" version="2.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.SignalR" version="1.2.2" targetFramework="net40" />
  <package id="Microsoft.AspNet.SignalR.Core" version="1.2.2" targetFramework="net40" />
  <package id="Microsoft.AspNet.SignalR.JS" version="1.2.2" targetFramework="net40" />
  <package id="Microsoft.AspNet.SignalR.Owin" version="1.2.2" targetFramework="net40" />
  <package id="Microsoft.AspNet.SignalR.SystemWeb" version="1.2.2" targetFramework="net40" />
  <package id="Microsoft.AspNet.Web.Optimization" version="1.1.3" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebApi" version="4.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebApi.Client" version="4.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebApi.Client.zh-Hans" version="4.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebApi.Core" version="4.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebApi.Core.zh-Hans" version="4.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebApi.WebHost" version="4.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebApi.WebHost.zh-Hans" version="4.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebPages" version="2.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.AspNet.WebPages.zh-Hans" version="2.0.30506.0" targetFramework="net40" />
  <package id="Microsoft.Net.Http" version="2.0.20710.0" targetFramework="net40" />
  <package id="Microsoft.Net.Http.zh-Hans" version="2.0.20710.0" targetFramework="net40" />
  <package id="Microsoft.Owin.Host.SystemWeb" version="1.0.1" targetFramework="net40" />
  <package id="Microsoft.Web.Infrastructure" version="1.0.0.0" targetFramework="net40" />
  <package id="Newtonsoft.Json" version="6.0.4" targetFramework="net40" />
  <package id="Owin" version="1.0" targetFramework="net40" />
  <package id="WebGrease" version="1.6.0" targetFramework="net40" />
</packages>
```

Mvc漏洞安全公告:https://technet.microsoft.com/zh-cn/library/security/ms14-059
Mvc补丁地址:https://www.microsoft.com/zh-CN/download/details.aspx?id=44533
Mvc4发布说明:https://docs.microsoft.com/en-us/aspnet/whitepapers/mvc4-release-notes
Mvc4安装包:https://www.microsoft.com/zh-CN/download/details.aspx?id=30683
