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
我们现在是还没有创建Home组件的，现在去项目下的components文件夹下新建一个Home.vue文件，把原有的HelloWorld.vue删除。

##### Home.vue我们分开一点一点讲：
1. 首先是头部，简单的使用了mint-ui的header控件，很简单的控件，在template添加如下代码:
    ```
    <mt-header title="Cy音乐助手">
      <mt-button icon="more" slot="right" @click="showActionSheet" style="margin-right:5px;"></mt-button>
    </mt-header>
    ```
    title是header的标题，名为right的插槽显示的是一个按钮，它的icon图标是mint的名为more的icon，另外我加了一点样式让它右边添加一点间距。
2. 搜索框我本来打算使用mint-ui的search组件的，但是他会有一个search-list，就是展现搜索结果列表的，他会占据一大块屏幕，我试了之后发现怎么都修改不到它的css样式把它高度设置成0，所以我就下载了AUI的素材（具体可以百度AUI），解压后把css文件夹中的aui.css和一个字体文件放到本项目中src目录下的assets目录下，同时引入样式文件，在vue文件的style标签中添加这段代码就可以引入了：
    ```
    @import '../assets/css/aui.css';
    ```
    然后在template中添加以下代码，做成搜索框:
    ```
    <div class="aui-searchbar" id="search">
      <div class="aui-searchbar-input aui-border-radius">
        <i class="aui-iconfont aui-icon-search"></i>
        <input type="search" placeholder="请输入搜索内容" v-model="formData.input">
      </div>
    </div>
    ```
    可以看到输入框绑定了formData数据中的input的属性，这个属性是告诉后台输入的搜索依据的值（后台搜索依据有三种），formData这个使我们要提交到后台的所有数据。（后面详细介绍）
3. 接着就是radiogroup，本来也是打算使用mint-ui的radio组件，奈何他只能是竖排，我又改不到它样式，无奈，继续使用AUI的，在template中继续添加一下代码:
    ```
    <div class="radioGroup">
      <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="netease"> 网易</label>
      <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="qq"> QQ</label>
      <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="kugou"> 酷狗</label>
      <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="kuwo"> 酷我</label>
      <label class="radio"><input class="aui-radio" type="radio" name="demo1" v-model="formData.type" value="xiami"> 虾米</label>
    </div>
    ```
    可以看到，每个radio绑定的都是formData中的type属性，这个属性是用来告诉后台希望从哪个平台搜索。接口是我从网上抓取的，有好多平台可以选，但是我就选了常用的五个出来，对应的值请看radio的value。
4. 接着就是按钮了，这个按钮用的是mint-ui的button组件，这个组件使用起来非常简单，直接上代码：
    ```
    <mt-button
      type="primary"
      @click.native="search"
      class="searchBtn">
      点击搜索音乐
    </mt-button>
    ```
    type可以控制按钮类型，primary这个代表主要按钮，颜色是蓝色（其他可以看官网详解），然后为这个按钮绑定了一个点击事件search代表发起搜索，这个方法在后面我们跟着formData一起介绍。
5. 接着就是下拉刷新上拉加载列表，这个使用的是mint-ui的loadmore组件，这个组件我用的时候碰到不少坑，好好介绍一下，先看看布局代码，在template中添加：
    ```
    <div class="aui-content" :style="'height:'+wrapperHeight+'px;overflow: scroll;'">
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
    ```
    代码比较长，我来讲解一下。
    
    首先最外面是一个class为aui-content的div，就是aui中的容器，另外有一个引人注目的字段wrapperHeight，这个是计算出来的高度，即获得手机屏幕高度减去它上面所有组件的可是高度，以此让这个容器填满这个高度。接着容器里面是mint-ui的loadmore组件，这才是主角。我们来看看。首先来看看第一个坑autoFill，让我们先来看看官网对这个属性的描述：loadmore 在初始化时会自动检测它的高度是否能够撑满其容器，如果不能则会调用 bottom-method，直到撑满容器为止。如果不希望使用这一机制，可以将 auto-fill 设为 false。一开始并没有注意到这么一段话，每次进页面的时候疯狂调用loadMore方法（上拉加载更多的方法）。
    
    topMethod代表的是下拉刷新执行的方法；bottomPullText是上拉是显示的文字；bottomDropText是上拉到一定距离显示的文字；bottomMethod是上拉加载更多执行的方法；bottom-all-loaded代表上拉加载是否已经全部加载完毕，如果为真代表没有更多的数据了，这时候就不会出发上拉加载更多的方法了。

    接着就是列表了，循环了songList这个数据来循环添加单个列表项，每个列表项都是一个mint-ui的cell-swipe组件，这个组件可以实现向左滑动右边有按钮出现，但是它不能放一个图标或者图片在左边，所以我就在左边加了一个图片标签，宽高写死。添加之后发现不会再一行，又加了inline-block样式。这里我们主要讲讲cell-swipe这个组件的属性。title是这个列表项的标题，label是列表项标题下面一行更小的文字，right是右边的按钮，滑动才会显示出来，传入的是一个数组。数组中：content是按钮的文字，style是按钮的样式，handler是按钮被点击时候出发的方法，同时我们会发现上面的代码我们还传入了一个index来判断点击的是第几个列表项。
6. 布局的最后是一个actionsheet，相当于是一个底部弹出的菜单，使用方法非常简单，首先在template中添加以下代码：
    ```
    <mt-actionsheet
      :actions="actions"
      v-model="sheetVisible">
    </mt-actionsheet>
    ```
    actions是菜单的列表项数据，一个数组；然后这个组件绑定了sheetVisible数据，这是一个布尔值，为真时候底部菜单会显示，还有一个进场动画过度，为假时就消失。让我们看看actions的值：
    ```
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
    ]
    ```
    我这里暂时只传入了菜单项的名称，还没传入被点击时候的响应方法。
7. 接下来就介绍一下formData和search方法，先来看看formData的数据格式：
    ```
    formData: {
      input: '',
      type: 'netease',
      filter: 'name',
      page: 1
    }
    ```
    type是音乐搜索来源平台，可以有的值就是前面说的radio绑定的5个value；<br>
    filter代表搜索依据，就是说靠什么来搜索歌曲，可以靠歌曲名称，歌曲id，歌曲官网链接,我们这里只做根据歌名来搜索；<br>
    input前面说了是搜索依据的值，如果filter是那么，那么input的值就应该是歌名；<br>
    page很明显是第几页的数据，这在我们上拉加载更多里面会用到。<br>
    接下来看看search方法（敲黑板），首先上代码：
    ```
    search () {
      let that = this
      that.formData.page = 1
      Indicator.open('努力搜索中...')
      getMusicList(this.formData, function (res) {
        that.songList = res.data
        Indicator.close()
      })
    }
    ```
    慢着，不对啊，getMusicList和Indicator是哪里来的？？？问得好！<br>
    Indicator是mint-ui的一个loading动画，就是界面上显示一个圆圈转啊转然后下面一串字请稍等那样的。对于getMusicList这个方法是我封装的，我在src目录下新建了一个api文件夹，文件夹里面又建了一个recommend.js，内容如下:
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
    我们引入了jQuery，调用了ajax发送请求（what？用jQuery发起请求？大材小用了吧！怎么不用axios？额，我还没学），然后对外输出getMusicList方法，这个方法传入两个参数，一个是data，我们传入前面说的formData就好了，另一个是成功的回调函数，失败的我就没写了。另外有一个很重要的点要说一下：接口是我抓取的，也就是别的网站的，也就是要访问不同源的资源，也就会产生跨域问题。按理来说跨域可以直接用jsonp解决，但是请注意看上面代码的请求方式是post，我抓取的这个接口要post请求，但是众所周知，jsonp只支持get请求。那么怎么解决呢？我用的是后端代理的方式。打开config目录下的index.js，修改proxyTable这一项的值如下：
    ```
    proxyTable: {
      '/api': {  //使用"/api"来代替"http://www.qmdai.cn" 
        target: 'http://www.qmdai.cn', //源地址 
        changeOrigin: true, //改变源 
        pathRewrite: { 
          '^/api': 'http://www.qmdai.cn' //路径重写 
          }
      }
    },
    ```
    这样就完美解决了跨域的问题。之后我们看看服务端给我们返回的数据是什么样的：
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
    我们把返回的data这个数组赋值给songList然后遍历循环，然后把单个数据填到对应位置就可以把结果列表展示出来了，如果不知道怎么把数据填充到对应位置的好好学习一下Vue基础吧。
#### 未完待续，明天接着把一个小问题跟大家说一下