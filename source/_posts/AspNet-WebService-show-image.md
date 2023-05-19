---
title: 在WebService中预览图片
date: 2017-11-21 14:22:00
tags:
  - Asp.Net
  - 开发笔记
categories:
  - .Net
  - Asp.Net
toc: false
---
项目中有这样一个需求，在通过请求第三方WebService接口的结果中，包含一个图片内容为base64的字符串。
由于经常需要查看图片内容，于是在考虑能不能在WebService的方法测试中提供传入一个图片base64的参数然后预览图片的功能。
分析了一下WebService的调试页面的请求和返回结果，返回的内容是一个xml结构。既然能够返回xml那么问题就好办了。
html也是xml。只要构造特定的html就可以了。于是第一版本的预览是这样的。构造html,body中构造一个img标签用base64显示图片。
```
	//版本1 使用html预览显示
	[WebMethod]
	public void Base64ToImage(string @base64)
	{            
		var result = string.Format(@"<!DOCTYPE HTML>
<html>
<head>
<meta charset='UTF-8'>
<title></title>
</head>
<body>
<img src='data:image/png;base64,{0}'/>
</body>
", @base64);

		var context = HttpContext.Current;
		if (context == null || context.Response == null)
		{
			return;
		}
		
		context.Response.Clear();
		context.Response.ClearHeaders();
		context.Response.Write(result);
		context.Response.End();
	}
```

但是考虑到img标签中用base64字符串来显示图片，在低版本浏览器的兼容性问题以及渲染问题。更进一步的想到直接向http写入输出流的方式来显示图片。
```
	//版本2 直接输出image类型
	[WebMethod]
	public void Base64ToImage(string @base64)
	{
		byte[] data = Convert.FromBase64String(@base64);
		var context = HttpContext.Current;

		if (context == null || context.Response == null)
		{
			return;
		}

		context.Response.Clear();
		context.Response.ClearHeaders();
		context.Response.ContentType = "image/jpeg";
		context.Response.BinaryWrite(data);
		context.Response.End();
	}
```
