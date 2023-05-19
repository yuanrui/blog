---
title: 解决Asp.Net配置异常杂记
date: 2016-10-18 14:06:07
tags:
  - Asp.Net
  - 开发笔记  
categories:
  - .Net
  - Asp.Net
---
### 相关配置数据无效，节点被锁定
问题描述：在Win10 x64位 IIS 10环境中，部署Asp.Net Mvc 3站点时，报“HTTP 错误 500.19 - Internal Server Error
无法访问请求的页面，因为该页的相关配置数据无效。”
配置错误：不能在此路径中使用此配置节。如果在父级别上锁定了该节，便会出现这种情况。锁定是默认设置的(overrideModeDefault="Deny")，或者是通过包含 overrideMode="Deny" 或旧有的 allowOverride="false" 的位置标记明确设置的。 
![错误描述 0x80070021](https://cloud.githubusercontent.com/assets/3859838/19508346/56305450-960b-11e6-9539-c99aa0f24650.png)

解决办法
1、在控制面板中打开"程序和功能"；
2、在程序和功能中，打开"启用或关闭Windows功能"；
3、选择"Internet Infomation Services" -> "应用程序开发功能"；
4、排除"CGI"选项
![Windows功能选项](https://cloud.githubusercontent.com/assets/3859838/19508410/b80f7cdc-960b-11e6-8ff1-c5fc82f10807.png)
参考链接：http://stackoverflow.com/questions/9794985/iis-this-configuration-section-cannot-be-used-at-this-path-configuration-lock
