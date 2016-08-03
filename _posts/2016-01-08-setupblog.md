---
layout:     post
title:      "博客环境搭建"
subtitle:   "在windows 搭建一个github博客"
date:       2016-01-08 0:30:00+0800
author:     "New"
header-img: ""
tags:
    - 环境搭建
    - blog
---

在中国、 在windows上搭建这些blog真是 麻烦

# 重要提示
在windows环境下，按照这个说明搭建出来的blog，会由于一些时区问题，造成blog无法生成或者时间解析错误
问题的原因和解析可以参照[这里](https://github.com/jekyll/jekyll/issues/1069)
在中国 使用windows系统 使用jekyll+github写博客的朋友如果没有此问题，可以留下你的ruby版本号

## 步骤：

 1. 搭建github博客
 2. 安装jekyll
 3. 发布第一篇文章
 
## 搭建github博客

这一步在百度上搜索会有很多的帖子，所以不在写了。

这一步结束，应该就可以在浏览器访问自己的博客首页了。

但是，在我做完这一步的时候， 我根本不知道接下来，怎么做，大部分文章都到此为止了。 
关键的问题，以什么形式写博客，我不知道。
翻看github上代码，发现都是前端代码，难道让我直接提交前端代码吗。。 这不可能啊
后来看到[如何利用GitHub建立个人博客：之一](http://my.oschina.net/nark/blog/116299)这篇帖子，
才算明白。原理是 github提供服务器，托管博客站点，同时还提供jekyll工具，
把markdown文件转化为相应的页面，所以自己只需要写markdown格式的博客即可。

为了方便查看博客的最终效果，肯定本地也要按照jekyll工具，可惜那篇帖子的楼主没有写续篇。

## 安装jekyll

安装过程参考[jekyll的说明](http://jekyllrb.com/docs/windows/#installation)，
其中的链接有些需要翻墙，才能流畅访问。

其中安装ruby的下载过程很慢，建议百度搜下别人分流的安装包，包括DevKit。

安装jekyll时，先设置下[国内代理](https://ruby.taobao.org/)不然也是慢的不行

## jekyll命令

    $ jekyll build
    # => The current folder will be generated into ./_site

    $ jekyll build --destination <destination>
    # => The current folder will be generated into <destination>

    $ jekyll build --source <source> --destination <destination>
    # => The <source> folder will be generated into <destination>

    $ jekyll build --watch
    # => The current folder will be generated into ./_site,
    #    watched for changes, and regenerated automatically.

    $ jekyll serve
    # => A development server will run at http://localhost:4000/
    # Auto-regeneration: enabled. Use `--no-watch` to disable.

    $ jekyll serve --detach
    # => Same as `jekyll serve` but will detach from the current terminal.
    #    If you need to kill the server, you can `kill -9 1234` where "1234" is the PID.
    #    If you cannot find the PID, then do, `ps aux | grep jekyll` and kill the instance. [Read more](http://unixhelp.ed.ac.uk/shell/jobz5.html).

## 发布第一篇文章

要发布文章了，但是排版也不能太丑，所以搜了jekyll模板，使用了这个clean主题。
然后就是按照jekyll的说明，修改下主题的相关配置，以添加自己的信息

从开始到写这篇文章，整了3+小时，主要原因是不熟悉用markdown写博客生成网页的流程，
不知道github上到底怎么添加一个博客。还有就是下载ruby安装包很慢。。

## 遇到的一些其他问题

 - 文章的样式要自己调整的，这个不是程序员还真弄不来，本来以为主题里这些应该都写好了，
   却发现基本是只满足demo文章的样式。。。
 - 文件名， category 都不要用中文，不然会出错，以后看看能不能解决
 - 最后提交时，出现 样式丢失的情况，主要原因是 baseUrl 设置了 /new-xd， 应该是还缺少其他改动
   先把baseUrl恢复为默认""

## 附加参考
[markdown的语法说明](http://wowubuntu.com/markdown/)

