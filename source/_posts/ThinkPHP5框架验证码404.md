---
title: ThinkPHP5验证码提示404问题解决
date: 2018-03-14 08:56:29
categories: PHP
tags: [问题解决记录]
---
最近在学习php，用的是ThinkPHP的框架，学习的时候碰到一个验证码刷新的时候的404问题，困扰了我很久始终没有解决，最终在php中文网的ThinkPHP5的学习视频课程中的讨论区找到解决办法。
首先看一波图片（出现的现象及报错图片）：
<!-- more -->
![正常情况](https://upload-images.jianshu.io/upload_images/8654767-9c66cbcd9a0b51f5)  
（这里可以看到验证码正常显示）

![出现404问题了](https://upload-images.jianshu.io/upload_images/8654767-0927b4eff57e1d99)  
（点击看不清换一张图片就没了）  

![控制台出现报错](https://upload-images.jianshu.io/upload_images/8654767-6dd568e7eeaef5db)  
（控制台出现报错，404）

这时候我的验证码刷新代码是这样的：  
![原代码](https://upload-images.jianshu.io/upload_images/8654767-9a71a6f9a63c8510)
查到的解决办法，将代码改成如下图这样：  
![解决问题的第一种办法](https://upload-images.jianshu.io/upload_images/8654767-6c35baa18b187eff)  
![解决问题的第二种办法](https://upload-images.jianshu.io/upload_images/8654767-cfe42d0215937825)  
还有人说可以修改php.ini文件，但是php我也是刚学习不久，所以没去深究，反正问题是解决了的，哈哈。  
当然解决了是不够的，还要用博客把它记下来才行啊！