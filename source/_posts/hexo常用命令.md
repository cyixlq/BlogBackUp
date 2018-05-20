---
title: hexo常用命令
date: 2018-03-14 08:56:29
categories: 随笔
tags: [随笔]
---
- 最近闲来无事，把我github上的个人博客重新捣鼓了一下，之前也研究过，部署了一个个人博客在我的github上，但是发现不是那么回事，不是我     理解的那个样子，最后只好重新折腾一下。
- 重新查找资料发现github上的个人博客一般是用**hexo+github pages**做出来的，于是乎，我就想用这篇博文记载一下hexo的一些常用命令。
- 新建一篇博文或一个文件 **hexo n** "文件名" 或者完整写法 **hexo new** "文件名"
- 发布 **hexo p** 或者完整写法 **hexo publish**
- 生成静态页面，也就是普普通通的网页，用于浏览的 **hexo g** 或者完整写法 **hexo generate**
- 启动本地服务器，用于预览生成的静态文件 **hexo s** 或者完整写法 **hexo server**
- 部署到服务器（一般我用来把生成的静态文件部署到github）**hexo d** 或者完整写法 **hexo deploy**
<!-- more -->
### 下面是一些快捷命令 ###
- **hexo generate --deploy 和 hexo deploy --generate ：** 先生成静态文件后部署到服务器（github），简写 ：**hexo d -g**
- 直接用简写命令 **hexo s -g** 可以生成静态文件并且启动本地服务器进行预览

后记
----------
- 部署hexo博文教程可以参考这篇基础博客[https://www.cnblogs.com/fengxiongZz/p/7707219.html](https://www.cnblogs.com/fengxiongZz/p/7707219.html "点我前往查看博文（基础）")，写的挺详细的，我就不重复造轮子了，然后是一篇进阶的，作者是同一人[http://www.cnblogs.com/fengxiongZz/p/7707568.html](http://www.cnblogs.com/fengxiongZz/p/7707568.html "点我前往查看博文（进阶）")
- 本地服务器默认端口是4000，所以访问部署到本地服务器的地址是**localhost:4000**，如果端口被占用，或者你不喜欢这个端口号，可以通过命令**hexo server -p 端口号** 例如：**hexo server -p 5000**