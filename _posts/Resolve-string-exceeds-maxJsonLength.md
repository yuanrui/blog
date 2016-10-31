---
title: 解决Asp.Net Mvc下Json字符串的长度超过了maxJsonLength的异常
date: 2016-05-18 17:12:57
tags:
  - 开发笔记
categories:
  - Mvc
  - .Net
  - Asp.Net
toc: false
---
在Mvc 3项目中老是出现“使用 JSON JavaScriptSerializer 进行序列化或反序列化时出错。字符串的长度超过了为 maxJsonLength 属性设置的值。”的异常信息。
查询Google之后找到解决方案如下：

```C#
public ContentResult GetResult()
{
    var obj = new object();
    var serializer = new JavaScriptSerializer { MaxJsonLength = Int32.MaxValue, RecursionLimit = 3 };
    
    var jsonResult = new ContentResult()
    {
        Content = serializer.Serialize(obj),
        ContentType = "application/json"
    };

    return jsonResult;
} 
```
另外还可以通过自定义JsonReuslt的方式解决这个异常, 具体做法参考这个链接：https://brianreiter.org/2011/01/03/custom-jsonresult-class-for-asp-net-mvc-to-avoid-maxjsonlength-exceeded-exception/
