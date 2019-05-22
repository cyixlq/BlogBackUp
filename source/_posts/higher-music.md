---
title: higher-music
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
![Top榜单及Top歌曲](https://user-gold-cdn.xitu.io/2019/1/14/1684a5581fb2a97d?w=1240&h=530&f=png&s=359598)

![歌曲搜索以及热搜关键词](https://user-gold-cdn.xitu.io/2019/1/14/1684a558208f362c?w=1240&h=533&f=png&s=76409)

![歌单详情](https://user-gold-cdn.xitu.io/2019/1/14/1684a55821124253?w=1240&h=533&f=png&s=325479)

![播放列表](https://user-gold-cdn.xitu.io/2019/1/14/1684a55821b2edb9?w=1240&h=530&f=png&s=86496)

### 特色功能 —— 歌单外链的使用方法：(已经从此项目中独立出来，[点我前往查看](https://cyixlq.top/2019/01/30/iframe/))
除了像一个正常网页音乐播放器外，还支持一个个人认为比较牛逼的功能 —— 歌曲外链，使用方法如下：
1. 登录网页QQ音乐，选择你需要制作外链的歌单，点击分享，点击复制链接：

![复制分享歌单链接](https://user-gold-cdn.xitu.io/2019/1/14/1684a558221f7fe4?w=1120&h=608&f=png&s=197192 "QQ20190114-094339.png")

2. 将复制好的链接粘贴到任何可以编辑的地方，然后将链接中的数字部分复制下来：

![复制链接中的数字部分](https://user-gold-cdn.xitu.io/2019/1/14/1684a5587ffcaa4d?w=812&h=229&f=png&s=156535 "QQ20190114-094532.png")

3. 在你博客中需要接入外链的地方加入以下代码(请注意，将下面链接中的`2947517062`替换成你上一步中复制的数字，如果不需要播放器自动播放请将下面`true`改成`false`，iframe的宽高你可以自定义，Chrome可能会禁止iframe内音频自动播放)：
    ```
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height=450 src="//cyixlq.gitee.io/music/#/iframe/2947517062/true"></iframe>
    ```
4. 效果图如下：

![效果图](https://user-gold-cdn.xitu.io/2019/1/28/16893a5bad2b32f9?w=1126&h=514&f=png&s=65303)

### 到这里，你的音乐外链就制作完成，注意并不是所有歌曲的播放地址都能解析出来，还望谅解，如果你喜欢本项目的话或者本项目对你有一定帮助的话，可以扫描下方二维码进行捐赠，以此来维持服务器的运转：

![捐赠](https://user-gold-cdn.xitu.io/2019/1/14/1684a5587a6187dc?w=200&h=654&f=png&s=66952 "erweima.png")