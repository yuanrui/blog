---
title: VS Code 上手杂记
date: 2017-02-09 17:22:00
toc: false
---
最近在把玩.net core, 顺便学习下用VS Code开发。
记录下开发步骤
1、安装.net core sdk. 地址:https://www.microsoft.com/net/download/core
2、下载并安装VS Code, 地址:https://code.visualstudio.com/
3、验证dotnet
安装完.net core sdk后，在命令行中用dotnet -version进行验证
![netcore-version](https://cloud.githubusercontent.com/assets/3859838/22683546/550e8700-ed53-11e6-95e6-5fb53147826a.png)
正常情况下会显示对应的版本号
4、创建dotnet项目
使用dotnet new -t Console命令创建控制台项目
![create project](https://cloud.githubusercontent.com/assets/3859838/22685417/319175e6-ed5b-11e6-9cd0-3fa1b016f07e.png)
创建完成后，对应目录会出现生成的Program.cs和project.json文件
同时执行dotnet restore命令
![dotnet restore](https://cloud.githubusercontent.com/assets/3859838/22685747/81ee7a4c-ed5c-11e6-92ef-c37f3ccba1d2.png)
5、调试项目
**备注**：如果是第一使用VS Code调试代码，还需要下载OmniSharp和.NET Core Debugger，注意观看控制台中的提示和右下角的Downloading packages. 下载安装完成后才可进行后续调试。提示：下载过程中不要关闭VS Code.
![vs code debug error](https://cloud.githubusercontent.com/assets/3859838/22776610/429d51fc-eeeb-11e6-9729-55af72b860ea.png)

使用Ctrl+Shift+D进行调试项目，会发现默认没有配置
![debug project](https://cloud.githubusercontent.com/assets/3859838/22685590/ddd9b8f4-ed5b-11e6-992f-07ac2eb914d2.png)
选择.NET Core创建配置，创建完成后还需要对配置进行修改。eg."program": "${workspaceRoot}/bin/Debug/netcoreapp1.1/Test.dll",
![change config](https://cloud.githubusercontent.com/assets/3859838/22685825/c9f3c996-ed5c-11e6-9ac7-17d7eaec219b.png)
修改完成相关配置后，可进行调试，出现经典的Hello World! 
![hello world](https://cloud.githubusercontent.com/assets/3859838/22776454/ab1f08f2-eeea-11e6-9450-8b7c2e0b56f4.png)

done!



