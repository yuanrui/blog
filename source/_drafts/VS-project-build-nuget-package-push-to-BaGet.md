---
title: VS项目生成nuget包发布到BaGet
date: 2022-03-08 15:40:32
tags:
  - 开发笔记
toc: true
---
### 前言

项目多了后，





```shell
del $(ProjectDir)\bin\Release\*.nupkg 
dotnet pack $(ProjectFileName) -c Release -o $(ProjectDir)\bin\Release --no-build
dotnet nuget push $(ProjectDir)\bin\Release\*.nupkg -s http://localhost:9099/v3/index.json -k yuanrui.key --no-service-endpoint --skip-duplicate
```



```shell
/c del /s /q *.nupkg && dotnet pack $(ProjectFileName) -c Release -o $(ProjectDir)\bin\Release && dotnet nuget push $(ProjectDir)\bin\Release\*.nupkg -s http://localhost:9099/v3/index.json -k yuanrui.key --no-service-endpoint --skip-duplicate
```

