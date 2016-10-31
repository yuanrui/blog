title: 如何使用Hexo
tags:
  - 资料文档
categories:
  - Hexo
date: 2016-05-13 13:39:00
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/zh-cn/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

### Quick Start

#### Create a new post

``` bash
$ hexo new "My New Post"
```
用new命令创建文章时，会将中间空格替换为横杠-,上述命令最终生成文件名为："My-New-Post.md". 创建时可以不用包含双引号，生成的文件取的最后一个单词，例如：hexo new how to use google，命令生成的文件为：google.md.
创建时最好使用英文，英文相对中文来说生成的Url更友好些。
More info: [Writing](https://hexo.io/docs/writing.html)

#### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

#### Clean

```
$ hexo clean
```
clean命令用于清除publish文件中已经生成的文件和清除缓存文件。
#### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

#### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

#### 安装插件
安装Rss插件 
``` bash
npm install hexo-generator-feed --save
```
安装github-card插件
``` bash
npm install hexo-github-card
```

参考：
http://www.jianshu.com/p/ba76165ca84d