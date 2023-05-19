---
title: 为Asp.Net Mvc项目的Js和Css添加版本号处理
date: 2016-08-10 18:22:00
tags:
  - Asp.Net
  - 开发笔记
categories:
  - .Net
  - Asp.Net
  - Mvc
---
### 为什么需要版本号处理
在项目开发部署过程中，经常会修改样式和脚本文件。在部署时经常会因为浏览器缓存的原因造成加载的样式和执行的脚本还是以前的版本。
解决浏览器缓存最常用的办法是，在服务器端渲染生成资源链接时，加上一个版本号，用于刷新客户端缓存。这个版本号只在第一次请求时重新加载资源文件，在后续的请求中从浏览器缓存中获取文件。
版本号相对来说是固定的，不是随机生成的。随机生成的版本号会造成每次加载页面时都会去服务器端请求数据，这不是我们想要的效果。
### 实现方式
生成版本号的方式有几种：
1、从配置文件中读取版本号
2、计算当前文件的hash值，用hash值作为版本号
3、使用程序集（DLL）发布时间作为版本号
第1种，发布部署时可以灵活控制，但是也容易忘记修改版本号，导致版本还是使用的以前的版本。也许可以通过自动发布的方式，自动修改版本号（没有尝试过）。
第2种，如果使用计算hash值的方式来处理，文件每次请求时都会去计算hash值。每次请求都计算hash有点消耗资源，改进办法是将路径和计算后的值放大缓存中，先判断hash值是否存在，不存在则计算并加入到缓存中。
第3种，可以将版本值的计算设计为静态类，并在静态类的静态构造方法中初始化静态变量，静态构造方法只需执行一次。每次发布替换DLL后，静态类会重新初始化。相比1、2种方式，灵活而不失简便。
```C#
public static class VersionUtils
{
    public readonly static DateTime VersionDate;

    public readonly static Int32 VersionNumber;

    static VersionUtils()
    {
        VersionDate = System.IO.File.GetLastWriteTime(typeof(VersionUtils).Assembly.Location);
        VersionNumber = Int32.Parse(VersionDate.ToString("yyyyMMddHHmm"));
    }
}

public static class HtmlHelperExtension
{
    public static MvcHtmlString Script(this HtmlHelper html, string contentPath)
    {
        return VersionedContent(html, "<script src=\"{0}\" type=\"text/javascript\"></script>", contentPath);
    }

    public static MvcHtmlString Style(this HtmlHelper html, string contentPath)
    {
        return VersionedContent(html, "<link href=\"{0}\" rel=\"stylesheet\" type=\"text/css\">", contentPath);
    }

    private static MvcHtmlString VersionedContent(this HtmlHelper html, string template, string contentPath)
    {
        contentPath = UrlHelper.GenerateContentUrl(contentPath, html.ViewContext.HttpContext) + "?v=" + VersionUtils.VersionNumber;
        return MvcHtmlString.Create(string.Format(template, contentPath));
    }
}

public static class UrlHelperExtension
{
    public static string ContentVersioned(this UrlHelper urlHelper, string contentPath)
    {
        return String.Format("{0}?v={1}", urlHelper.Content(contentPath), VersionUtils.VersionNumber);
    }

}
```
### 使用方式
```C#
<link href="@Url.ContentVersioned("~/Content/Site.css")" rel="stylesheet" type="text/css" />
@Html.Style("~/Content/bootstrap.css");
@Html.Script("~/Scripts/angular.js");
```