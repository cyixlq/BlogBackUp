---
title: 使用Vue开发Cy音乐助手网页应用版(一)——首页页面完成
date: 2018-05-28 10:10:13
tags: Vue实战
categories: Vue
---
### 前言
在不久前，我做过一款APP叫做Cy音乐助手的。可以实现音乐搜索，在线试听音乐，试听时歌词显示，收费音乐下载，歌词下载等功能。有兴趣的小伙伴可以安装一下试试看，下载地址:https://fir.im/cymusicuu ,目前只有安卓端。如今浅浅学习了一下Vue忍不住手痒痒想把它做成个网页版应用，心动不如行动，让我们开始吧！
<!-- more -->
### 正文
在看接下来的文章之前，我默认各位是安装好了Vue所需要的所有环境和IDE。博主用的IDE是vs code，大家可以根据自己喜好安装自己喜欢的IDE。好了，话不多说，开始了。

首先打开命令行，执行命令: vue init webpack "vue_cy_music"，vue_cy_music是我的项目名，大家可以根据自己喜好来命名。接着命令行就会让你填一些选项，比如项目名，作者信息，项目描述等。这些根据个人情况填写就好了。后面的一些比如是否使用ESLint，博主选择了是，是否启用什么测试插件的我全部选的否，因为我不懂什么测试的。。。

项目构建好之后我们先安装mint-ui框架，方便我们快速搭建页面。这里先介绍一下mint-ui：mint-ui框架是饿了么公司出品的一个移动端的框架，可以实现快速搭建页面，具体使用我们可以到:http://mint-ui.github.io/#!/zh-cn 查看。

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
      <mt-header title="Cy音乐助手">
        <mt-button icon="more" slot="right" @click="showActionSheet" style="margin-right:5px;"></mt-button>
      </mt-header>
      <mt-search style="height:auto;" v-model="formData.input"></mt-search>
      <div class="radioGroup">
        <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="netease"> 网易</label>
        <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="qq"> QQ</label>
        <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="kugou"> 酷狗</label>
        <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="kuwo"> 酷我</label>
        <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="xiami"> 虾米</label>
      </div>
      <mt-button
        type="primary"
        @click.native="search"
        class="searchBtn">
        点击搜索音乐
      </mt-button>
      <div :style="'height:'+wrapperHeight+'px;overflow: scroll;'">
        <mt-loadmore
          :autoFill="false"
          :topMethod="refresh"
          bottomPullText="上拉加载更多"
          bottomDropText="释放加载"
          :bottomMethod="loadMore"
          :bottom-all-loaded="bottomAllLoaded"
          ref="loadmore">
            <div v-for="(item,index) in songList"
                 :key="index">
              <img :src="item.pic" width="48" height="48" style="display: inline-block">
              <mt-cell-swipe
                :title="item.title"
                :label="item.author"
                :right="[
              {
                content: '试听',
                style: { background: 'red', color: '#fff' },
                handler: play(index)
              }]"
                style="display: inline-block;width: 85%">
              </mt-cell-swipe>
            </div>
        </mt-loadmore>
      </div>
      <mt-actionsheet
        :actions="actions"
        v-model="sheetVisible">
      </mt-actionsheet>
  </div>
</template>

<script type="text/ecmascript-6">
import { getMusicList } from '../api/recommend'
import { Indicator } from 'mint-ui'
export default {
  data () {
    return {
      formData: {
        input: '',
        type: 'netease',
        filter: 'name',
        page: 1
      },
      wrapperHeight: 0,
      sheetVisible: false,
      actions: [
        {
          name: 'Cy出品,必属精品'
        },
        {
          name: '使用说明'
        },
        {
          name: '关于此应用'
        },
        {
          name: '设置'
        }
      ],
      songList: [], // 歌曲结果列表
      bottomAllLoaded: false // 上拉加载时数据是否全部加载完毕
    }
  },
  mounted () {
    this.wrapperHeight = window.screen.height - 170
  },
  methods: {
    showActionSheet () {
      this.sheetVisible = !this.sheetVisible
    },
    search () {
      let that = this
      that.formData.page = 1
      Indicator.open('努力搜索中...')
      getMusicList(this.formData, function (res) {
        that.songList = res.data
        Indicator.close()
      })
    },
    loadMore () {
      let that = this
      that.formData.page++
      getMusicList(that.formData, function (res) {
        if (res.data.length < 10) {
          this.bottomAllLoaded = true
          // 如果当页返回的歌曲结果列表长度小于10就代表没有更多结果了，
          // 这时候就把bottomAllLoaded置为true
        }
        for (let i = 0; i < res.data.length; i++) {
          that.songList.push(res.data[i])
        }
        that.$refs.loadmore.onBottomLoaded()
      })
    },
    refresh () {
      let that = this
      that.formData.page = 1
      getMusicList(this.formData, function (res) {
        that.songList = res.data
        that.$refs.loadmore.onTopLoaded()
      })
    },
    play (index) {
    }
  }
}
</script>

<style scoped>
@import '../assets/css/aui.css';
div.mint-search-list{
  height: 0px;
}
.radioGroup{
    padding: 5px 2%;
}
.radio{
    display: inline-block;
    width: 19%;
}
.searchBtn{
    width: 96%;
    margin: 10px 2%;
    height: 40px;
    background-color: #26a2ff;
    color: aliceblue;
    font-size: 10px;
}
</style>
```
Home组件做好了，让我们先看看效果吧！现在的效果图如下所示：

![home.gif](https://upload-images.jianshu.io/upload_images/8654767-f164ab968dc3d42f.gif?imageMogr2/auto-orient/strip)

上面的代码还涉及到一些音乐搜索逻辑，让我来给大家详细介绍一下这部分的代码吧！

从GIF来看，从头到脚介绍一下的我做法：
1. 首先是头部，直接使用的mint-ui的mt-header控件，中间是标题，右侧是一个图标。
2. 然后下面的搜索框也是直接使用的mint-ui里面的搜索框。
3. 接下来是radio按钮组，由于mint-ui的radio按钮组无法排列成横排，或者说是我不会改，我就用的aui的，引入aui的css和字体放到assets目录下，在style标签内加上@import进行引入。
4. 接下来的上拉加载更多，下拉刷新使用的是mint-ui的loadMore控件，可侧滑列表用的是cell swipe，图片我试了下插槽插在icon位置的话图片是在列表上方，效果不好，还是自己用了个div把两个包起来，然后做了些样式调整。
5. 努力搜索中这个弹出框是Indicator控件

大致就这么些东西，详细用法看看mint-ui文档就能轻易写出来的。

接下来说说后端数据的获取，我在项目目录下新建了一个api文件夹，在里面有一个函数，代码如下：
```
import $ from 'jquery'

export function getMusicList (data, success) {
  $.ajax({
    url: 'api/yinyuesou/',
    type: 'post',
    data: data,
    dataType: 'json',
    headers: {
      'accept': 'application/json, text/javascript, */*; q=0.01'
    },
    success: success
  })
}
```
从上面代码发现，我们引入了jQuery，用他的ajax方法想api的路径发起了一个请求，实际上我们请求的接口是： http://www.qmdai.cn/yinyuesou/ ，但是我们运行的域名是localhost:8080，很明显存在跨域问题。本来跨域问题可以使用jsonp解决，但是众所周知，jsonp只支持get请求，是不支持post的。然后最可恨的是，我抓取的接口需要post请求。这时候我就只能配置后端代理来解决这个问题了。在config目录下，index.js文件中的proxyTable的值进行如下配置:
```
proxyTable: {
  '/api': {  //使用"/api"来代替"http://www.qmdai.cn" 
    target: 'http://www.qmdai.cn', //源地址 
    changeOrigin: true, //改变源 
    pathRewrite: { 
      '^/api': 'http://www.qmdai.cn' //路径重写 
      }
  }
}
```
从上面代码中我们可以看到使用api路径替换了原来的请求路径，所以当我们使用ajax请求api路径时，后端对我们的请求进行转发，从而解决了这个问题。

可以看到我们使用ajax发起post请求，设置dataType为json，也就是我们期望服务器给我们返回json数据，然后data我们通过Home组件传过来的，主要有四个数据：
1. type为字符串，代表音乐来源，原网站音乐来源选项挺多的，我这里只拿出来五个常见的来源。当type为netease代表在网易云音乐中搜索歌曲，qq代表qq音乐来源，kugou代表酷狗音乐，kuwo代表酷我音乐，xiami代表虾米音乐。
2. page为数字型，代表请求第几页数据。
3. filter为字符串，代表依据歌曲什么属性进行搜索，原网站支持根据名称搜索、歌曲id搜索，歌曲链接搜索。我这里只需要根据名称搜索，所以filter为固定值name。
4. input为字符串，代表歌曲属性的值，例如我这里固定依据歌曲名搜索，所以input的值代表歌曲名。

方法另外需要传入一个回调函数，用过ajax的都知道，服务器响应的数据会在回调函数的形参中，我们打印一下发现我们请求到的json数据如下（部分）：
```
{
  code: 200,
  data: [
    {
      "type": "netease",
      "link": "http:\/\/music.163.com\/#\/song?id=190499",
      "songid": 190499,
      "title": "给你们",
      "author": "张宇",
      "lrc": "这里是歌词，太长了，剪了",
      "url": "http:\/\/music.163.com\/song\/media\/outer\/url?id=190499.mp3",
      "pic": "http:\/\/p1.music.126.net\/otCQ-2wUgLaEN2_W8Nf_DA==\/57174604656249.jpg?param=300x300"
    },
    ...这里省略九条歌曲信息
  ],
  "error": ""
}
```
这串json数据相当简单，code是状态码，200代表成功；error代表错误信息；data就是返回的数据，总共返回10条歌曲信息。data中单条数据type代表歌曲来源，link是官网音乐链接，songid是歌曲ID，title是歌曲名，author是歌手，lrc是歌词，url是解析出来的音乐真实地址，pic是歌曲图片封面。搞明白这些之后我们把对应属性填入到列表相应位置（详细请看代码）通过v-for把歌曲列表遍历出来。
#### 未完待续，明天接着把一个小问题跟大家说一下