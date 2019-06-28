---
title: 博客音乐歌单外链
date: 2019-01-30 13:03:21
tags: [vue, 音乐播放, 博客音乐外链, vue实战]
categories: Vue
---
#### 话不多说，直接先来看效果图：
{% qnimg 博客音乐歌单外链效果图.png title:博客音乐歌单外链效果图 alt:博客音乐歌单外链效果图 %}`

<!-- more -->

#### 功能特色：
0. 支持QQ音乐，网易云音乐歌单外链
1. 仿网易云歌单外链（包括功能）

#### 使用方法：
0. 需要传入的参数有三个依次是type，id，autoplay
1. 首先确定你要制作的歌单外链的平台，暂时只支持QQ音乐，网易云音乐这两个音乐平台的歌单。如果是QQ音乐type取值tencent，网易云音乐取值netease。
2. 获取你歌单的ID：
    - QQ音乐：
        1. 登录网页QQ音乐，选择你需要制作外链的歌单，点击分享，点击复制链接：
        {% qnimg 复制分享歌单链接.png title:复制分享歌单链接 alt:复制分享歌单链接 %}`

        2. 将复制好的链接粘贴到任何可以编辑的地方，然后将链接中的数字部分复制下来(这串数字就是你歌单的ID)：
        {% qnimg 复制分享链接中的数字部分.png title:复制分享链接中的数字部分 alt:复制分享链接中的数字部分 %}`
    - 网易云音乐：
        1. 登录网页网易云音乐，点击你创建的歌单：
        {% qnimg 点击网易云音乐中你创建的歌单.png title:点击网易云音乐中你创建的歌单 alt:点击网易云音乐中你创建的歌单 %}`
        2. 打开歌单之后，将浏览器上的地址中数字部分复制下来，这就是你网易云音乐歌单的ID（PS：虽然网易云音乐支持生成外链歌单，但是需要会员或者付费的歌曲在歌单里面整个歌单是无法生成外链的；不过本歌单外链工具可以）：
        {% qnimg 获取网易云音乐歌单ID.png title:获取网易云音乐歌单ID alt:获取网易云音乐歌单ID %}`

3. 将以下代码加到你的博客中，把type和id替换成你上面获取的值，另外链接中true代表自动播放，false表示不自动播放。另外需要知道，Chrome和Safari会禁用自动播放属性，设置成true可能也不会自动播放，在别的浏览器上可能可以：

    ```
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height=450 src="//cyixlq.gitee.io/iframe/#/type/id/true"></iframe>
    ```
    例如：
    ```
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height=450 src="//cyixlq.gitee.io/iframe/#/tencent/2947517062/true"></iframe>
    ```

#### 致此，一个歌单外链就制作完成！