---
title: Android MVVM探索(二) - DataBiding常用注解
date: 2018-10-22 16:32:38
tags: [MVVM]
categories: Android
---
### 本来想直接一章节大致讲讲完DataBinding的，但是要知道，DataBinding知识点还是蛮多的（我也还没全部了解完）。加上我又喜欢写的细一点，所以一篇写下来洋洋洒洒一长篇，所以还是分开来写吧！今天（好吧，其实和上一篇是同一天）我们要说的DataBinding常用的一些注解！

<!-- more -->
## Android MVVM探索系列
[Android MVVM探索(一) - DataBiding初解]("http://cyixlq.top/2018/10/22/Android MVVM探索(一) - DataBiding初解/")

### 6， 一些常用注解说明
- @BidingAdapter
    
    在xml文件中，总是有些控件的属性不够我们用，比如我们有一张图片的网络地址，我们想直接在xml中将地址绑定到ImageView，让它显示这张图片，很明显的ImageView自带的属性明显不能完成我们的要求。那怎么办呢？我们需要拓展属性！
    首先我们新建一个名为ViewBindingAdapters的Kotlin文件，（如果是Java请新建一个普通类，类名随意），内容如下：
    ```
    package top.cyixlq.test

    import android.databinding.BindingAdapter
    import android.widget.ImageView
    import com.bumptech.glide.Glide

    @BindingAdapter("imgUrl")
    fun setImgUrl(view: ImageView, url: String) {
        Glide.with(view).load(url).into(view)
    }

    // Java如下：
    public class ViewBindingAdapters {
        @BindingAdapter("imgUrl")
        public static void setImgUrl(ImageView view, String url) {
            Glide.with(view).load(url).into(view)
        }
    }
    ```

    可以看到，我们使用了Glide，所以需要在app的build.gradle文件中dependencies下加入(我写这篇文章时，最新的Glide版本为4.8.0):
    ```
    implementation 'com.github.bumptech.glide:glide:4.8.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
    ```

    我们需要在网上找了一张图片的链接，并且把它加入到MainViewModel的属性中:
    > val imgUrl = "这里是图片地址，地址太长，我就不贴在这里"

    我们在activity_main.xml文件中加入一个ImageView，并使用我们拓展的属性：
    ```
    <ImageView
        android:layout_width="80dp"
        android:layout_height="80dp"
        imgUrl="@{viewModel.imgUrl}"/>
    ```

    最后别忘了在manifest中添加网络权限：
    ```
    <uses-permission android:name="android.permission.INTERNET" />
    ```

    那么，这个注解是如何判断是给哪个控件添加的拓展属性呢？请看本小节开头我们创建的那个Kotlin文件的内容，我们用@BindingAdapter注解修饰了一个setImgUrl的方法，注解中包含一个字符串“imgUrl”。方法的第一个形参的类型是ImageView，所以，这个注解是通过第一个形参的类型来判断拓展的是哪个控件的属性，第一个形参也是设置了该拓展属性的控件的实例。第二个形参传进来的是一个String类型的值，也就是在xml布局文件中传进来的viewModel.imgUrl，然后我们在方法体中进行一下处理即可实现自动加载网络图片了！

    @BindingAdapter注解需要传入的第一个参数是字符串数组。如果你只想拓展一个属性，那么只传一个字符串就行。如果你想拓展多个属性，那么你就需要传入一个字符串数组，一个字符串就代表一个拓展属性。当数组长度大于一时，我们就需要设置它的第二个参数，requireAll（中文意思：全部必须）。这个参数为布尔型。当为true时(默认就是为true)，就像他的中文意思一样，全部需要，代表你传入的第一个参数，字符串数组的拓展属性必须在一个控件上全部要设置，否则的话你想设置哪个拓展属性就设置哪个拓展属性！示例代码如下：
    ```
    // 当为false时
    @BindingAdapter(value = ["imgUrl", "bgRes"], requireAll = false)
    fun setImgUrl(view: ImageView, url: String, res: Int) {
        Glide.with(view).load(url).into(view)
        view.setBackgroundResource(res)
    }
    <!-- xml文件代码 -->
    <ImageView
        android:layout_width="80dp"
        android:layout_height="80dp"
        imgUrl="@{viewModel.imgUrl}"/>
    
    // 当为true时
    @BindingAdapter(value = ["imgUrl", "bgRes"], requireAll = true)
    fun setImgUrl(view: ImageView, url: String, res: Int) {
        Glide.with(view).load(url).into(view)
        view.setBackgroundResource(res)
    }
    <!-- xml文件代码 -->
    <!-- 这段请放在data标签中 -->
    <import type="top.cyixlq.test.R"/>
    
    <ImageView
        android:layout_width="80dp"
        android:layout_height="80dp"
        imgUrl="@{viewModel.imgUrl}"
        bgRes="@{R.mipmap.ic_launcher}"/>
    ```

    如果第二个参数设置为true但是在xml中有没有将全部的拓展属性设置好的话在编译的时候就会报错：
    > Found data binding errors.

    最后看看这个注解的源代码，更方便理解：
    ```
    @Target(ElementType.METHOD)
    public @interface BindingAdapter {
        String[] value();
        boolean requireAll() default true;
    }
    ```