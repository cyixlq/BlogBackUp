---
title: higher-music 基于Vue的支持歌单外链的在线网页音乐播放器
date: 2019-01-14 10:38:58
tags: [vue, 音乐播放, 博客音乐外链, vue实战]
categories: Vue
---
一款基于Vue打造的网页在线音乐播放器，利用工作空闲时间与大学同学[@ganp1020](https://github.com/ganp1020)一起开发。目前正在开发阶段，已经能正常使用。**支持歌单外链这一特色功能！**

<!-- more -->

### 项目演示地址：
- http://cyixlq.gitee.io/music/

### 实现的功能：
- 上一曲
- 下一曲
- 暂停/播放
- 音质选择
- 遇到没有的音质自动切换音质源
- 音乐搜索
- 播放列表展示
- 正在播放歌曲标志
- 歌单全部播放
- 歌单，歌曲top榜显示
- 同时适配桌面端和移动端

### 项目技术：
- vue
- vue-router
- vuex
- axios
- jsonp
- [Vuetify](https://vuetifyjs.com/zh-Hans/)

### 项目截图
![Top榜单及Top歌曲](/images/higher-music1.png)
![歌曲搜索以及热搜关键词](/images/higher-music2.png)
![歌单详情](/images/higher-music3.png)
![播放列表](/images/higher-music4.png)

### 特色功能 —— 歌单外链的使用方法：(已经从此项目中独立出来，[点我前往查看](https://cyixlq.top/2019/01/30/iframe/))
除了像一个正常网页音乐播放器外，还支持一个个人认为比较牛逼的功能 —— 歌曲外链，使用方法如下：
1. 登录网页QQ音乐，选择你需要制作外链的歌单，点击分享，点击复制链接：
![复制分享歌单链接](/images/复制分享歌单链接.png)

2. 将复制好的链接粘贴到任何可以编辑的地方，然后将链接中的数字部分复制下来：
![复制分享链接中的数字部分](/images/复制分享链接中的数字部分.png)


3. 在你博客中需要接入外链的地方加入以下代码(请注意，将下面链接中的`2947517062`替换成你上一步中复制的数字，如果不需要播放器自动播放请将下面`true`改成`false`，iframe的宽高你可以自定义，Chrome可能会禁止iframe内音频自动播放)：
    ```
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height=450 src="//cyixlq.gitee.io/music/#/iframe/2947517062/true"></iframe>
    ```
4. 效果图如下：
![外链歌单效果图](/images/外链歌单效果图.png)

### 到这里，你的音乐外链就制作完成，注意并不是所有歌曲的播放地址都能解析出来，还望谅解，如果你喜欢本项目的话或者本项目对你有一定帮助的话，可以扫描下方二维码进行捐赠，以此来维持服务器的运转：
![服务器捐赠](/images/服务器捐赠.png)