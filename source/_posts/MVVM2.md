---
title: Android MVVM探索(二) - DataBiding常用注解
date: 2018-10-22 16:32:38
tags: [MVVM]
categories: Android
---
### 本来想直接一章节大致讲讲完DataBinding的，但是要知道，DataBinding知识点还是蛮多的（我也还没全部了解完）。加上我又喜欢写的细一点，所以一篇写下来洋洋洒洒一长篇，所以还是分开来写吧！今天（好吧，其实和上一篇是同一天）我们要说的DataBinding常用的一些注解！

[本篇文章代码地址](https://github.com/cyixlq/MVVMTest)

<!-- more -->
## Android MVVM探索系列
[Android MVVM探索(一) - DataBiding初解](https://cyixlq.top/2018/10/22/MVVM1/)

[Android MVVM探索(二) - DataBiding常用注解](https://cyixlq.top/2018/10/23/MVVM2/)

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

    补充一下，传入的属性值前面可以加上命名空间，就像下面那样（命名空间有一定规范,最好是英文单词）：
    ```
    @BindingAdapter(value = ["app:imgUrl", "app:bgRes"], requireAll = true)
    fun setImgUrl(view: ImageView, url: String, res: Int) {
        Glide.with(view).load(url).into(view)
        view.setBackgroundResource(res)
    }
    ```

    如果你加入了命名空间，相应的你也需要在xml布局文件中引入命名空间并且加上去，就像下面那样:
    ```
    <!-- 引入app命名空间 -->
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto">

    <!-- 控件相应属性加上命名空间 -->
    <ImageView
        android:layout_width="80dp"
        android:layout_height="80dp"
        imgUrl="@{viewModel.imgUrl}"
        app:bgRes="@{R.mipmap.ic_launcher}"/>
    ```

    最后看看这个注解的源代码，更方便理解：
    ```
    @Target(ElementType.METHOD)
    public @interface BindingAdapter {
        String[] value();
        boolean requireAll() default true;
    }
    ```
- @BindingConversion

    这个注解的作用就是将不符合某个控件属性的值的类型转换成符合的类型。举个例子：TextView的text属性是需要String类型的，假如我们传入时间戳(Long类型)很明显是不行的，但是我就是不想自己每个都自己手动转换一下，怎么办？那么这个注解就派上用场了。我们直接上代码，就以TextView这个例子来说明：
    
    1.我们新建一个Kotlin文件，文件名为ViewBindingConversions，文件内容如下：
    ```
    @SuppressLint("SimpleDateFormat")
    @BindingConversion
    fun convertIntToString(value: Long): String {
        val formatter = SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
        return formatter.format(value)
    }
    ```

    2. 把我们之前布局文件中新加入一个TextView，就像下面的代码，之后编译运行，我们发现可以正常运行。
    ```
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{System.currentTimeMillis()}"/>
    ```
    值得注意的是，这个注解，声明一个，所有控件都会自动转换。也就是说我们上面的那个方法，凡是某个属性需要String类型的，如果你传入了Long，它就会通过方法体中的代码进行转换，最后填充的属性值就是返回的结果。加入Button的text你也传入了Long，那么也会进行转换，并不只限于TextView。用这个注解修饰的方法只有一个形参，它的类型代表是xml中传进来的值的类型，最后函数必须要一个返回值，返回匹配控件支持的值得类型。所以感觉就是这个注解用起来不会太灵活！

- @InverseMethod

    在开发中，我们经常遇到某个字符串，或者某个数字代表某种状态。比如0代表女孩子，1代表男孩子。但是用户选好男还是女之后，我们回传到后台的数据应该是0或1，但是我们不想手动转换怎么办，那就用这个注解。这个注解的作用和上面那个注解@BindingConversion有点类似，但是不同的是，这个可以具体作用到某个控件实例上，还有双向绑定的作用。如果不知道双向绑定是什么意思，请参照上一小章节。首先还是新建一个Kotlin文件，文件名为：ViewInverseMethods，Java的话还是和上面一样新建一个类，类名随意。文件内容如下：
    ```
    // 这个注解参数为反转的方法名，意味着一个这个注解需要两个方法才能完成
    @InverseMethod("sexToNum")
    fun numToSex(num: Int): String {
        return when (num) {
            0 -> "女"
            1 -> "男"
            else -> "未知性别"
        }
    }

    fun sexToNum(sex: String): Int {
        return when(sex) {
            "女" -> 0
            "男" -> 1
            else -> 2
        }
    }
    ```

    是的，因为这个注解设计到双向绑定，有转换过去的方法肯定,肯定就有转换回来的方法。我们在MainViewModel中添加属性：
    ```
    // 因为涉及双向绑定，这里必须是可观察的数据类型
    val num = ObservableInt(1)
    ```
    
    在布局文件中添加一个TextView和一个EditText，TextView用来观察num这个值的变化，EditText用来展示转换好之后的数据：
    ```
    <!-- 这里别忘了导入，导入了下面的EditText才能用这个类 -->
    <import type="top.cyixlq.test.ViewInverseMethodsKt"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{String.valueOf(viewModel.num)}"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@={ViewInverseMethodsKt.numToSex(viewModel.num)}"/>
    ```
    这时候编译运行，EditText直接显示了男，如果我们删除男，EditText立马显示了未知性别。因为空字符串会对应到sexToNum中的else，所以num的值瞬间变成2，而2又对应numToSex中的else，所以EditText就会显示未知性别。当我们把未知性别几个字全部删除，输入1，或者2也好，num都是2，因为字符串-"1",字符串-"2"都是对应sexToNum方法中的else。但是如果我们在输入框中输入男，num就变成1了，输入女num就变成0了。其中的过程我就不继续详细解释了。下面贴出一张动态图：
    ![InverseMethod注解详解动图.gif](https://upload-images.jianshu.io/upload_images/8654767-386d9ceaa25b82d8.gif?imageMogr2/auto-orient/strip)

### 暂时我就用到这么些注解，我认为比较常用的。其他注解我在下面贴上链接，大家可以去看看。另外本章节中出现的问题还望大家留言指正，毕竟还是在MVVM探索道路中！

[DataBinding使用教程（三）：各个注解详解](https://blog.csdn.net/qiang_xi/article/details/75379321)

[DataBinding最全使用说明](https://juejin.im/post/5a55ecb6f265da3e4d7298e9#heading-19)
