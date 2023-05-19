---
title: 'Javascript常用验证函数收集'
date: 2016-11-21 10:40:00
tags:
  - 开发笔记
  - JavaScript
categories:
  - JavaScript
toc: false
---
字符串是否为空或为空格
```
function (value) {
	return value == null || /\S/.test(value);
} 
```

邮箱地址格式验证
```
function (value) {
	//https: //html.spec.whatwg.org/multipage/forms.html#valid-e-mail-address
	///^[a-zA-Z0-9.!#$%&'*+\/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/

	//form 163
	return !/^[._-]?(?:[a-z0-9]+[._-]?)+[a-z0-9]?@[a-z0-9](?:[.-]?[a-z0-9]+)*$/i.test(value)
}
```

车辆号牌格式
```
function (value) {
	return /(^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼]{1}[A-Z0-9]{6}$)|(^[A-Z]{2}[A-Z0-9]{2}[A-Z0-9\u4E00-\u9FA5]{1}[A-Z0-9]{4}$)|(^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼]{1}[A-Z0-9]{5}[挂学警军港澳]{1}$)|(^[A-Z]{2}[0-9]{5}$)|(^(08|38){1}[A-Z0-9]{4}[A-Z0-9挂学警军港澳]{1}$)/.test(value);
}
```

手机号码
```
function (value) {
	var length = value.length;
	var re = /^(((13[0-9]{1})|(15[0-9]{1})|(18[0-9]{1}))+\d{8})$/;
	return (length == 11 && re.test(value));
}
```

计算字符串长度
```
function (value) {
	var utfStr = value.match(/[^\x00-\xff]/ig);
	return value.length + (utfStr == null ? 0 : utfStr.length);
}
```