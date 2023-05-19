---
title: 谨慎使用系统换行符
date: 2021-08-19 18:10:01
tags:
  - 开发笔记
toc: false
---

项目中有一个计算Http请求哈希值的环节，客户端按照指定文本格式生成哈希值；服务端收到Http请求后按照相同格式的生成文本并计算哈希，比较哈希值的异同判断请求信息是否被篡改。定义的文本格式如下：

```
Request Method      + "\n" +
Accept              + "\n" +  
Content-Type        + "\n" +
Request URI         + "\n" +
MD5(Request Body)   + "\n" +
jti                 + "\n" +
iat
```

公司内部测试没啥问题，移交给客户后报出生成的哈希值和工具生成的哈希值不匹配，而用工具生成的哈希值是可以请求服务接口的。截图也看了，日志也看了，明明是相同的文本，为啥生成的哈希就是不一样呢。找了好久才发现是不可见字符的影响，原来在服务端拼接指定格式文本的时候使用的StringBuilder的AppendLine方法，而AppendLine方法的内部自动追加了一个Environment.NewLine.

我们都知道“\n”是换行符，一直以来C#的各种编程实践都推荐使用 Environment.NewLine 用于指定系统换行符，但是 **Environment.NewLine != "\n"**. 所以在这个示例中正确的用法应该是使用Append方法，在Append方法里添加"\n"作为换行使用.

| 操作系统                       | 回车换行所用字符                                             |
| ------------------------------ | ------------------------------------------------------------ |
| Linux/Unix/Android/Mac OS X 10 | \n = Newline = 0x0A = 10 = LF =Line Feed = 换行 = Ctrl + J   |
| Mac OS 9                       | \r = Return = 0x0D = 13 = CR = Carriage Return = 回车 = Ctrl + M |
| Windows/Dos                    | \r \n = 0x0D 0x0A = CR LF = 回车 换行                        |

Windows平台的换行符是”\r\n“，而Linux平台的换行符是”\n“，在不同平台下调用系统换行符返回的换行符是不一样的。很多时候我们需要保证平台兼容性，让Linux平台和Windows平台的保持一致时应该拒绝使用编程语言提供的系统换行符，应该使用原生的"\n"或"\r\n"。至于是使用"\n"还是"\r\n"来定义换行格式，理论上都可以，只要约定好就行。

| 编程语言/环境 | 系统换行符                           |
| ------------- | ------------------------------------ |
| C#            | Environment.NewLine                  |
| Java/JDK7前   | System.getProperty("line.separator") |
| Java/JDK7后   | System.lineSeparator()               |
| Python        | os.linesep                           |



------



后记：在排查完问题后，很好奇到底还有那些字符属于不可见字符。原来在ASCII中存在控制字符和打印字符的说法，通常我们说的不可见字符大多是控制字符。常见的控制字符（US-ASCII控制字符）见下表。

| 十进制 | 十六进制 | 控制字符 | 转义字符 | 说明                                            |
| :----- | :------- | :------- | :------- | :---------------------------------------------- |
| 0      | 00       | NUL      | \0       | Null character(空字符)                          |
| 1      | 01       | SOH      |          | Start of Header(标题开始)                       |
| 2      | 02       | STX      |          | Start of Text(正文开始)                         |
| 3      | 03       | ETX      |          | End of Text(正文结束)                           |
| 4      | 04       | EOT      |          | End of Transmission(传输结束)                   |
| 5      | 05       | ENQ      |          | Enquiry(请求)                                   |
| 6      | 06       | ACK      |          | Acknowledgment(收到通知/响应)                   |
| 7      | 07       | BEL      | \a       | Bell(响铃)                                      |
| 8      | 08       | BS       | \b       | Backspace(退格)                                 |
| 9      | 09       | HT       | \t       | Horizontal Tab(水平制表符)                      |
| 10     | 0A       | LF       | \n       | Line feed(换行键)                               |
| 11     | 0B       | VT       | \v       | Vertical Tab(垂直制表符)                        |
| 12     | 0C       | FF       | \f       | Form feed(换页键)                               |
| 13     | 0D       | CR       | \r       | Carriage return(回车键)                         |
| 14     | 0E       | SO       |          | Shift Out(不用切换)                             |
| 15     | 0F       | SI       |          | Shift In(启用切换)                              |
| 16     | 10       | DLE      |          | Data Link Escape(数据链路转义)                  |
| 17     | 11       | DC1      |          | Device Control 1(设备控制1) /XON(Transmit On)   |
| 18     | 12       | DC2      |          | Device Control 2(设备控制2)                     |
| 19     | 13       | DC3      |          | Device Control 3(设备控制3) /XOFF(Transmit Off) |
| 20     | 14       | DC4      |          | Device Control 4(设备控制4)                     |
| 21     | 15       | NAK      |          | Negative Acknowledgement(拒绝接收/无响应)       |
| 22     | 16       | SYN      |          | Synchronous Idle(同步空闲)                      |
| 23     | 17       | ETB      |          | End of Trans the Block(传输块结束)              |
| 24     | 18       | CAN      |          | Cancel(取消)                                    |
| 25     | 19       | EM       |          | End of Medium(已到介质末端/介质存储已满)        |
| 26     | 1A       | SUB      |          | Substitute(替补/替换)                           |
| 27     | 1B       | ESC      | \e       | Escape(溢出/逃离/取消)                          |
| 28     | 1C       | FS       |          | File Separator(文件分割符)                      |
| 29     | 1D       | GS       |          | Group Separator(分组符)                         |
| 30     | 1E       | RS       |          | Record Separator(记录分隔符)                    |
| 31     | 1F       | US       |          | Unit Separator(单元分隔符)                      |
| 32     | 20       | SP       |          | White space                                     |
| 127    | 7F       | DEL      |          | Delete(删除)                                    |



参考链接：

https://zh.wikipedia.org/zh-hans/%E6%8E%A7%E5%88%B6%E5%AD%97%E7%AC%A6

https://www.ruanyifeng.com/blog/2006/04/post_213.html

https://www.crifan.com/files/doc/docbook/char_encoding/release/webhelp/ascii_ctrl_char.html

https://www.crifan.com/detailed_carriage_return_0x0d_0x0a_cr_lf__r__n_the_context/