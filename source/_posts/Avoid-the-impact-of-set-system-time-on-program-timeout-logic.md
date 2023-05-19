---
title: 避免系统校时对程序超时判断逻辑的影响
date: 2021-09-05 15:00:58
tags:
  - 开发笔记
toc: false
---

程序的超时判断逻辑，一般用于指定时间范围内执行某项操作或等待操作结果。举个例子，Tcp通信中客户端向服务端发送请求数据后，需要等待服务端返回结果。这里的等待一般是有时间限制的，如果不做超时处理，客户端接收数据的线程将长时间处于阻塞状态，影响客户端的运行性能。
在超时处理中，有一种很常见的用法是有Bug的，比如下面的代码。

```c#
    var sock = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
    sock.ReceiveTimeout = 10000;
    sock.Connect("192.168.1.100", 10000);

    var startAt = DateTime.Now;
    var buffer = new Byte[4096];

    while ((DateTime.Now - startAt).TotalMilliseconds < sock.ReceiveTimeout)
    {
        if (!sock.Poll(10, SelectMode.SelectRead) || sock.Available <= 0)
        {
            Thread.Sleep(1);
            continue;
        }

        sock.Receive(buffer, SocketFlags.None);
    }
```
示例代码采用DateTime来获取系统时间，当前时间减去开始时间得到时间范围，当时间范围大于某个值时，超时。这个用法之所以存在Bug是因为是没有考虑系统校时的影响，在时钟回拨的时候，while循环有可能需要执行更长的时间。正确的用法应当采用System.Diagnostics命名空间的Stopwatch.
```c#
    var watch = Stopwatch.StartNew();
    while (watch.Elapsed.TotalMilliseconds < sock.ReceiveTimeout)
    {
        if (!sock.Poll(10, SelectMode.SelectRead) || sock.Available <= 0)
        {
            Thread.Sleep(1);
            continue;
        }

        sock.Receive(buffer, SocketFlags.None);
    }
```
那么，Stopwatch就不会受系统校时影响了吗？答案是看硬件支持情况。.NET Framework版的Stopwatch内部实现基于系统硬件支持高分辨率性能计数器（QueryPerformanceFrequency），不支持高分辨率性能计数器时采用DateTime.UtcNow.Ticks作为时间戳。

```c#
    //.NET Framework Stopwatch部分源代码
    static Stopwatch() {                       
        bool succeeded = SafeNativeMethods.QueryPerformanceFrequency(out Frequency);            
        if(!succeeded) {
            IsHighResolution = false; 
            Frequency = TicksPerSecond;
            tickFrequency = 1;
        }
        else {
            IsHighResolution = true;
            tickFrequency = TicksPerSecond;
            tickFrequency /= Frequency;
        }
	}

    public static long GetTimestamp() {
        if(IsHighResolution) {
            long timestamp = 0;    
            SafeNativeMethods.QueryPerformanceCounter(out timestamp);
            return timestamp;
        }
        else {
            return DateTime.UtcNow.Ticks;
        }   
    }
```

通过查看源代码我们知道，可以使用Stopwatch.IsHighResolution字段来判断计时器是否基于高分辨率性能计数器。在最新版的.Net Core源代码中，这个字段直接初始化为true，也就是说.Net Core默认都是采用高分辨率性能计数器来做时间戳的。为什么.Net Core中不像Framework那样内部做下兼容性判断呢，可能的原因是，在Windows XP及以后的操作系统都支持QueryPerformanceFrequency函数，再加上现在的硬件环境普遍越来越好，所以直接取消掉了兼容性处理。

总结，推荐使用Stopwatch来做超时判断处理。尽管一些老旧机器不支持高分辨率性能计数器，传统的.NET Framework在内部做了兼容性处理，.Net Core在最新的硬件上运行默认是支持的，可以放心使用。



参考链接：

https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/Diagnostics/Stopwatch.cs

https://docs.microsoft.com/zh-cn/dotnet/api/system.diagnostics.stopwatch.ishighresolution?view=net-5.0

https://docs.microsoft.com/zh-cn/windows/win32/sysinfo/acquiring-high-resolution-time-stamps

https://referencesource.microsoft.com/#System/services/monitoring/system/diagnosticts/Stopwatch.cs,c5257ce1ef4b0422

https://referencesource.microsoft.com/#System/compmod/microsoft/win32/SafeNativeMethods.cs,b692d974115ae110

https://docs.microsoft.com/zh-cn/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency
