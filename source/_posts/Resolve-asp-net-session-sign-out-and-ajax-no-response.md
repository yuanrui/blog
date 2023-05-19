---
title: 解决Asp.Net Session退出造成Ajax请求无响应的Bug
date: 2016-10-31 16:20:00
tags:
  - Asp.Net
  - 开发笔记
categories:
  - .Net
  - Asp.Net
  - Mvc
---
### 问题描述
在一般的Web后台管理系统的使用过程中，大多都是在单窗口中操作，用户所有的操作都在同一个标签页面中。
有时候会出现这种情况：
1、用户将当前页面的Url在新的标签页中打开；
2、在新的标签页面中注销当前登陆账户。
3、再返回当前页面进行操作（假设界面上的CRUD操作都是通过Ajax请求处理），这个时候的任何Ajax操作都不会返回正确的结果，如果不刷新页面，页面不会重定向到登陆页面；
解决问题的办法很简单，让用户刷新下页面就行。但是有时候用户是不知道需要刷新的。特别是打开后台管理系统，长期不操作界面，造成服务器端Session过期，需要用户重新登陆系统。
对于追求后台管理系统傻瓜式操作的追求来说，刷新页面并不是我们想要的结果。
### Ajax调用处理过程
浏览器发起Ajax请求后，正常情况下调用目标地址相关方法并返回结果。服务器端的Session过期或失效后，Ajax请求的地址将会被重定向到登陆页面，返回的结果是登陆页面的html。
### 解决方案
搞清楚Ajax调用处理过程后，解决问题的主要关键在于，如何判别返回的结果是不是登陆页面。如果返回的结果是登陆页面，则将window.location设置为登陆页的地址，否则不做处理。
一个简单的解决方案是在登陆页中添加一个特殊的Header. 同时在母版页中使用Jquery注册ajaxComplete事件，捕获之前设置的Header, 如果存在跳转到登陆页面，不存在则不做处理。
登陆页相关代码
```C#
[HttpGet, AllowAnonymous]
public ActionResult Login(string ReturnUrl)
{
	this.HttpContext.Response.AddHeader("__LoginUrl", "/Home/Login");
	ViewBag.ReturnUrl = ReturnUrl;
	
	return View();
}
```
母版页(_Layout.cshtml)中的Js事件处理
```javascript
<script src="https://code.jquery.com/jquery-1.12.4.min.js" type="text/javascript"></script>
<script type="text/javascript">
$(document).ajaxComplete(function(event, jqxhr, settings){
	if (jqxhr.status == 200) {
		var loginPageUrl = jqxhr.getResponseHeader("__LoginUrl");
		if(loginPageUrl && loginPageUrl !== ""){
			window.location.replace(loginPageUrl);
		}
		return;
	}
});
</script>
```
之前考虑过，使用ajaxError事件来处理跳转。因为实际使用过程中Ajax传输的数据大多都是Json，重定向到登陆页后，将Html转化为Json时会抛异常同时触发ajaxError事件。使用ajaxError对于ajaxComplete来说，只有Ajax有异常的情况下才处理，不用处理判断每次Ajax请求，相对效率高一点。
由于我们项目的弹窗表单都是采用的Ajax Get请求加载的，返回的是Html，不会触发ajaxError事件。权衡之后，还是使用ajaxComplete.

顺便说一下，这里也可以用Cookie来替代Header, 具体处理方式类似这里不再复述。唯一需要注意的是，如何控制Cookie过期的问题。

参考链接：http://stackoverflow.com/questions/199099/how-to-manage-a-redirect-request-after-a-jquery-ajax-call