---
title: php出现Noinput file specified问题解决
date: 2018-03-14 08:56:29
categories: PHP
tags: [问题解决记录]
---
学PHP学着学着又出问题了，这不，又过来记录记录。  
这次出现的问题叫做php出现Noinput file specified，大概问题我觉得是因为文件找不到。  
出现的起因是这样的，我做了一个后台首页判断，看看你是否登录，如果没有登录的话就要跳转到登录页面，于是它的跳转地址是这个：
```
http://www.cy.com/index/user/login.html
```
<!-- more -->
于是乎，页面就直接出现Noinput file specified这一串字符串，我看了一眼地址，原因很简单，我没有index.php在里面，如果输入下面的地址访问就是没问题的：
```
http://www.cy.com/index.php/index/user/login.html
```
可是总不能在用户未登录的时候跳转到登录页面又要用户自己输入index.php吧，况且用户还不一定知道要这么改，于是一番搜索，找到解决办法。我用的服务器是Apache的，解决办法就是打开ThinkPHP框架中的public目录（我用的是ThinkPHP框架），里面有个文件是 **.htaccess** ，我们把它打开，它现在的内容是这样的：
```
<IfModule mod_rewrite.c>
  Options +FollowSymlinks -Multiviews
  RewriteEngine On
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^(.*)$ index.php/$1 [QSA,PT,L]
</IfModule>
```
打开之后我们要做的很简单，看到那个index.php这串字符没？我们在index.php后面加上一个 **?** 就OK了，改完之后的代码如下：
```
<IfModule mod_rewrite.c>
  Options +FollowSymlinks -Multiviews
  RewriteEngine On
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^(.*)$ index.php?/$1 [QSA,PT,L]
</IfModule>
```
接着我重新访问一下我的首页，正常跳转到登录界面了，激动！！！又增加了点用户体验！！！

后记
---
上面的办法只对Apache服务器有效，其他修改办法参照下文：  
### (一)IIS Noinput file specified  
**方法一：** 改PHP.ini中的doc_root行，打开ini文件注释掉此行，然后重启IIS  
**方法二：** 请修改php.ini，找到 **; cgi.force_redirect = 1** 去掉前面分号，把后面的1改为0即
**cgi.force_redirect = 0**

### (二)nginx配置遭遇No inputfile specified
**方法一：** php.ini(/etc/php5/cgi/php.ini)的配置中这两项  
**cgi.fix_pathinfo=1** (这个是自己添加的)  
**doc_root=**  
**方法二：** nginx配置文件/etc/nginx/sites-available/default中注意以下部分  

```
location ~ .php$ {
fastcgi_pass 127.0.0.1:9000;
fastcgi_index index.php;
fastcgi_param SCRIPT_FILENAME /var/www/nginx-default$fastcgi_script_name;
include fastcgi_params;
}
```
上面的各项参数请按照你的项目实际情况配置