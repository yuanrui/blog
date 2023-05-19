---
title: C#特定格式字符串转换为DateTime
date: 2017-05-04 14:01:00
tags:  
  - .Net
  - 开发笔记
categories:
  - .Net
toc: false
---
将字符串"20170504112153"转换为DateTime类型时，调用DateTime.Parse方法，发现抛出FormatException：该字符串未被识别为有效的 DateTime。
难道只能将字符串转换为特定的格式，比如下面这样？
```
	var input = "20170504112153";
	DateTime time;
	
	if (input.Length > 13)
	{
		var timeInput = input.Substring(0, 4) + "-" + input.Substring(4, 2) + "-"
			+ input.Substring(6, 2) + " " + input.Substring(8, 2) + ":"
			+ input.Substring(10, 2);

		time = DateTime.Parse(timeInput);
	}
```
这种方式让人的感觉，不够优雅。有没有更好的方式？答案是有的，调用DateTime.ParseExact方法。
ParseExact方法可以指定DateTime类型的转换格式
```
	var input = "20170504112153";
	var time = DateTime.ParseExact(input, "yyyyMMddHHmmss", System.Globalization.CultureInfo.CurrentCulture);
```
但是，这个方法还是有局限性的，比如为input值加上毫秒时，再次转换时还是会抛出FormatException.
不过，ParseExact有一个重载的方法可以解决这个问题
public static DateTime ParseExact(string s, string[] formats, IFormatProvider provider, DateTimeStyles style);
参数formats表示：DateTime类型允许格式的数组，所以你可以这样写
```
	var input = "20170504112153999";
	var date = DateTime.ParseExact(input
		, new string[] { "yyyyMMddHHmmss", "yyyyMMddHHmmssf", "yyyyMMddHHmmssff", "yyyyMMddHHmmssfff" }
		, CultureInfo.CurrentCulture, DateTimeStyles.None);
```
将毫秒的每种可能格式列举出来，方法有点笨，这也不失为一种解决办法。