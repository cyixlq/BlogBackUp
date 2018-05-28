---
title: 使用Vue开发Cy音乐助手网页应用版(一)——首页页面完成
date: 2018-05-28 10:10:13
tags: Vue实战
categories: Vue
---
### 前言
在不久前，我做过一款APP叫做Cy音乐助手的。可以实现音乐搜索，在线试听音乐，试听时歌词显示，收费音乐下载，歌词下载等功能。有兴趣的小伙伴可以安装一下试试看，下载地址:https://fir.im/cymusicuu ,目前只有安卓端。如今浅浅学习了一下Vue忍不住手痒痒想把它做成个网页版应用，心动不如行动，让我们开始吧！

### 正文
在看接下来的文章之前，我默认各位是安装好了Vue所需要的所有环境和IDE。博主用的IDE是vs code，大家可以根据自己喜好安装自己喜欢的IDE。好了，话不多说，开始了。

首先打开命令行，执行命令: vue init webpack "vue_cy_music"，vue_cy_music是我的项目名，大家可以根据自己喜好来命名。接着命令行就会让你填一些选项，比如项目名，作者信息，项目描述等。这些根据个人情况填写就好了。后面的一些比如是否使用ESLint，博主选择了是，是否启用什么测试插件的我全部选的否，因为我不懂什么测试的。。。

项目构建好之后我们先安装vant ui框架，方便我们快速搭建页面。这里先介绍一下vant ui：vant ui框架是有赞公司出品的一个移动端的框架，可以实现快速搭建页面，具体使用我们可以到:https://www.youzanyun.com/zanui/vant#/zh-CN/intro 查看。

首先我们看看app.vue里面的内容：
```
<template>
  <div id="app">
    <!-- 这里本来是有一个logo图标，已经被我删除了 -->
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
/* 这里本来有一些多余样式也被我删除了 */
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
```
从代码中我们可以看到这里主要是用一个router-view来显示界面的，因此我们要去配置一下路由，打开项目下的router文件夹中的index.js文件，修改里面的内容：
```
import Vue from 'vue'
import Router from 'vue-router'
// 这里删除了一个引入的没用的组件HelloWorld，引入了一个我们自定义的组件Home
import Home from '@/components/Home'

Vue.use(Router)

export default new Router({
  routes: [
    // 根目录的路由改成了Home组件
    {
      path: '/',
      name: 'Home',
      component: Home
    }
  ]
})

```
我们现在是还没有创建Home组件的，现在去项目下的components文件夹下新建一个Home.vue文件，把原有的HelloWorld.vue删除。Home.vue中新添加如下内容:
```
<template>
  <div>
    <van-nav-bar
        class="nav"
        title= 'Cy音乐助手'>
        <van-icon class="likeIcon" name="like" slot="left" />
        <van-icon class="menuBtn" name="wap-nav" slot="right" />
    </van-nav-bar>
    <van-search
        v-model='musicName'
        show-action
        placeholder='请输入要搜索的音乐名称'
        @search='onSearch'>
        <div class="searchBtn" slot="action" @click="onSearch" >搜索</div>
    </van-search>
    <van-radio-group v-model="musicType" class="radioGroup">
        <van-radio name="1" class="radio">网易</van-radio>
        <van-radio name="2" class="radio">QQ</van-radio>
        <van-radio name="3" class="radio">酷狗</van-radio>
        <van-radio name="4" class="radio">酷我</van-radio>
        <van-radio name="5" class="radio">虾米</van-radio>
    </van-radio-group>
  </div>
</template>

<script type="text/ecmascript-6">
import jsonp from 'jsonp'
export default {
  data () {
    return {
      musicName: '',
      musicType: '1'
    }
  },
  methods: {
    onSearch () {
    }
  }
}
</script>

<style scoped>
    .nav{
        background-color: #3399ff;
        color: #fff;
    }
    .searchBtn{
        width: 60px;
        text-align: center;
    }
    .menuBtn{
        color: #fff;
    }
    .likeIcon{
        color: red;
    }
    .radioGroup{
        padding: 5px 8px;
        width: 100%;
    }
    .radio{
        display: inline-block;
        width: 18%;
    }
</style>
```