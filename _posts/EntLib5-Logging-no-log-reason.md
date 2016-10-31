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

LogEntry类用于封装传递需要记录的日志信息。LogEntry包含一个类型为IDictionary<string, object>的属性ExtendedProperties.使用时需要注意的是，字典的类型Value类型为object, 但建议往其中写入值时最好保存为string类型。因为每个Lintener都需要指定一个格式化器（Formatter），可以Formatter中指定显示内容，在显示ExtendedProperties中的Value值时可能不支持对应的object转换为string.