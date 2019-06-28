---
title: 使用Vue做一个购物车
date: 2018-05-19 13:27:52
categories: Vue
tags: [Vue实战, vue, 购物车]
---
#### 前言
本人刚大三，现在到了下学期，学校不让我们回去，赶我们出去实习。嗯。。。我是大专狗，不过一直热爱技术。只怪当初高中研究手机刷机包等技术荒废了学业（有点后悔了，文凭不好看了），其实很喜欢IT的，但是当时没电脑，Java都没入门。（好像闲话有点多）
#### 假正文
最近我实习的公司在做网上商城的一个项目，我负责购物车这一块。这个项目是一个Web项目，没有进行前后端分离，但是又得做手机端，感觉好像哪里不对。。。 
<!-- more -->
web框架使用的是SpringMVC，模板框架是FreeMarker，想到以后要做移动端，果断还是用json来进行数据交互，并没有用freemarker。网页静态文件全部写好了，放在了springmvc的Views中。按理来说还是进行前后端分离好点的，但是做网页的没接触过Vue，那好吧。。。
于是我就想到在页面直接引入Vue，可是又是在内网环境开发，只好在自己个人笔记本上下载vue.js再拷贝到内网电脑上进行页面上的引入。
#### 真正文
首先让我们看一下静态页面的效果图：![静态页面效果图](https://upload-images.jianshu.io/upload_images/8654767-b7bbde16f91309cf.png)
##### 简单说一下这个功能模块的需求：
1. 勾选全选，所有商品全部选中。在取消全选框的时候所有商品取消选择。
2. 点击单个商品上的加号减号进行数量的增加和减少，右边小计实时计算出这个商品的价格合计。
3. 点击单个商品上的删除按钮将商品从购物车中删除。
4. 底部已选实时显示已经勾选的商品，右边合计金额实时显示所有勾选的商品的小计之和。
(是的，需求看起来不多，但是要结合后台去做还是需要点功夫的，但是这篇文章我们不牵扯后台，在前台造数据)
##### 现在让我们开始吧
***一***，创建一个Vue对象，设置好数据。
```
var cart; //全局Vue对象
//通过封装一个方法来创建Vue对象
function createVue(list) {  //传入通过后台获取的list
  cart = new Vue({
	el:'#cart',
	data(){
		return {
			list:list  //商品列表
		}
	}
  });
}
```
***二***，假设从后台请求到数据，然后赋值到Vue对象中
```
window.onload = function () {
		//请求后台代码   。。。。
		//请求成功后将获得的list赋值给cart的list
		let list = [
			{
				goodsTitle: "卫龙辣条",						    //商品名
				specifications: "大包",						 //商品规格
				unitPrice: "5",								  //商品单价
				subimage1Filename :"20180317eeftyd.jpg",		//商品图片名
				purchaseQuantity: 6						//商品数量
			}, 
			{
				goodsTitle: "雕牌洗衣粉",
				specifications: "大包",
				unitPrice: "13",
				subimage1Filename: "20180317ggptfg.jpg",
				purchaseQuantity: 1
			}, 
			{
				goodsTitle: "旺仔牛奶",
				specifications: "20盒装",
				unitPrice: "45",
				subimage1Filename: "20180317feftyp.jpg",
				purchaseQuantity: 1
			}];
			createVue(list);  //执行创建Vue对象方法
	}
);
```
***三***，修改html部分代码，将数据展示出来
```
<tr v-for="(item,index) in list">
  <td>
	<input type="checkbox" :id="'check'+index" name="checkboxs" />
	<label :for="'check'+index"></label>
  </td>
  <td>
	<img :src="'路径前缀/'+item.subimage1Filename" />
  </td>
  <td style="text-align:left;">
	<p>{{item.goodsTitle}}</p>
	<p>规格：{{item.specifications}}</p>
  </td>
  <td>￥{{item.unitPrice}}</td>
  <td class="adddel">
	<em v-on:click="minius(index)">-</em>
	<input type="number" v-model.number="item.purchaseQuantity" />
	<em v-on:click="add(index)">+</em>
  </td>
  <td>￥{{item.unitPrice * item.purchaseQuantity}}</td>
  <td><button v-on:click="checkDel(index)">删除</button></td>
</tr> 
```
这样就能将单个商品部分全部循环打印出来，并且将对应的信息打印在对应位置。效果图如下：
![效果图，图中的图片名和路径是我编的，所以找不到](https://upload-images.jianshu.io/upload_images/8654767-ea638e93b0b0c21b.png)
***四***，实现全选和勾选时候总价的计算，这部分算是有点挑战了。我的思路是在Vue对象中新增加一个数据用来标识商品的选中状态，所以创建Vue方法中的代码改成如下所示：
```
cart = new Vue({
	el: '#cart',
		data() {
			return {
				list: list,
				checkeds: new Array(list.length) //初始化成list的长度
		}
	});
```
然后在html中将商品对应的checkbox与checkeds绑定起来,修改后的代码如下：
```
<input type="checkbox" :id="'check'+index" name="checkboxs" v-model="checkeds[index]" />
```
利用computed属性计算价格总和：
```
sum () {
	let sum = 0;
	for (let i in this.list) {
		if (this.checkeds[i])  //如果checkeds[i]的结果为truth，则进行累加
		  sum += this.list[i].unitPrice * this.list[i].purchaseQuantity;
	}
	return sum;
}
```
HTML部分，我们在对应位置用{{sum}}带入就能进行显示了。这样就能实现计算勾选过的商品小计之和了。接下来实现全选功能，在methods属性中添加一个方法checkAll，具体代码如下：
```
checkAll (event) {  //这里的event就是全选checkbox对象
	if (event.checked) {  //如果全选的checkbox选中，将checkeds所有的值设置为true，对应商品checkbox的选中状态自动更新
		for (let i = 0; i < this.checkeds.length; i++) {
			Vue.set(this.checkeds, i, true);
		}
	 else {  //否则就进行与上面相反的操作
		for (let i = 0; i < this.checkeds.length; i++) {
			Vue.set(this.checkeds, i, false);
		}
	}
}
```
经过上面的一波操作，已经可以实现全选和点选时候的价格之和计算。我们还要统计商品选中的数量，这个很简单，同样使用computed属性，对checkeds中结果为truth的进行统计就好了，代码如下：
```
checkNum: function () {
	let num = 0;
	for (let i in this.checkeds) {
		if (this.checkeds[i]) {
			num++;
		}
	}
	return num;
}
```
然后在html中的对应位置用{{checkNum}}代入即可。现在我们已经实现了近一半需求，让我们继续完成他们吧！
***五***，实现购物车物品单个删除功能，这个就很简单啦，我们在methods中增加一个del方法，使用js数组的splice方法就可以实现。
```
del (index) {
    this.list.splice(index, 1);  //只需要从数组中移除对应项，视图会自动更新，不得不说，Vue太棒啦！
    this.checkeds.splice(index,1); //同时删除对应的选中状态标识
}
```
然后就是给删除按钮绑定点击事件(index是循环列表时候的下标)：
```
<button v-on:click="del(index)">删除</button>
```
这样我们就轻松实现了删除单个商品的需求，当然防止用户误删，在用户点击删除按钮时我们可以弹出一个确认框提示用户，这里我们就不去实现了。
***六***，实现购物车单个商品的数量增加，减少，并实时更新商品的小计。首先在methods中添加增加方法add和减少方法minius：
```
add (index) {
	this.list[index].purchaseQuantity++;  //这里按理来说应该查询后台对应商品库存量来进行限制的，这里不涉及到后台所以没加
},
minius (index) {
	if (this.list[index].purchaseQuantity > 1) {  //这里添加一个限制，最少要有一个商品
		this.list[index].purchaseQuantity--;
	}
}
```
然后我们在对应的加和减的按钮上绑定事件来触发这两个方法（index为列表循环时候的下标）：
```
<td class="adddel">
	<em v-on:click="minius(index)">-</em>
	<input type="number" v-model.number="item.purchaseQuantity" />
	<em v-on:click="add(index)">+</em>
</td>
<td>￥{{item.unitPrice * item.purchaseQuantity}}</td>
```
从上面的代码可以看到我们在小计一栏直接进行商品单价和数量相乘，这样就可以实现实时更新了。

##### 至此，我们的需求就算是完成了，最后给大家留两个小问题思考一下
***一***，如何实现批量删除？
***二***，在全选之后，我们取消了一个商品的状态，全选框的选中状态仍然是选中的，此时应该是不选中的，或者当我们一个一个把商品的选中状态全部勾选，全选框的状态仍然是补选中的，此时应该是选中状态（如下两图所示），这个现象如何解决？
![问题二的现象一](https://upload-images.jianshu.io/upload_images/8654767-cdd1b5f82dfe7ff7.png)
![问题二的现象二](https://upload-images.jianshu.io/upload_images/8654767-aca0bb9e8f0ec645.png)

#### 后文
本文的所有代码已经托管到GitHub，如果本文代码有误，请以GitHub上的为准，GitHub地址：https://github.com/cyixlq/vue_shopping_cart