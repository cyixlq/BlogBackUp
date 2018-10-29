---
title: Android MVVM探索(一) - DataBiding初解
date: 2018-10-22 09:39:00
tags: [MVVM]
categories: Android
---
##### 发现我好久没写博客，其实最近一直都很想写博客的，但是不知道写点什么好。刚好碰上最近在学习Android的MVVM设计模式以及官方提供给我们的控件，所以才有了这篇文章。（其实还是因为我懒，我懒！）

今天我想给大家讲讲DataBinding，为了保证我写的不会出错，我也借鉴参考了不少文章和视频。给大家看看一篇我个人觉得还不错的。[DataBinding最全使用说明（掘金博客）](https://juejin.im/post/5a55ecb6f265da3e4d7298e9)还有某课网的视频，[Android Data Binding实战-入门篇](https://www.imooc.com/learn/719)、[Android Data Binding实战-高级篇](https://www.imooc.com/learn/720)

[本篇文章代码地址](https://github.com/cyixlq/MVVMTest)

<!-- more -->

[Android MVVM探索(一) - DataBiding初解](https://cyixlq.top/2018/10/22/MVVM1/)

[Android MVVM探索(二) - DataBiding常用注解](https://cyixlq.top/2018/10/23/MVVM2/)

[Android MVVM探索(三) - ViewModel，DataBinding，LiveData混合三打](https://cyixlq.top/2018/10/29/MVVM3/)

### 1, 什么是DataBinding？

DataBinding,2015年IO大会介绍的一个框架，字面理解即为数据绑定，是Google对MVVM在Android上的一种实现，可以直接绑定数据到xml中，并实现自动刷新（即，数据变化UI进行相应的变化）。而且还支持一些表达式。比如常见的三元运算符：
> 1+x == 3 ? "true" : "false"

它还可以支持lambda表达式：

> (v,fcs) -> presenter.onFocusChange(user)}

使用了DataBinding，可以省去一些控件绑定代码，例如：findviewById等。

### 2, 开始使用DataBinding

要想使用DataBinding的话，首先要在你安卓工程中，安卓Application的module（一般为app这个module）的android配置中加上如下代码：
```
android{
    // 这里省去一些常有的配置代码
    dataBinding {
        enabled = true;
    }
}
```

另外，如果你是使用Kotlin进行编程的话，你还要在加入了上面代码的Gradle文件中顶部加上以下代码，否则Kotlin将无法识别DataBinding资源，至于什么是DataBinding资源，我们后面会提到：
```
apply plugin: 'kotlin-kapt'
```

是的，你没有看错，就这么简单我们就加上了DataBiding，不需要引入任何依赖。
首先我们先建立一个普通类作为ViewModel：
```
package top.cyixlq.test

class MainViewModel {
    var name = "张三"
    var age = 15
    var isMan = true
    fun log() {
        Log.d("MyTAG", "按钮被点击了一下")
    }
}
```

其次，我们要将我们布局文件代码进行一些改动。
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="viewModel"
            type="top.cyixlq.test.MainViewModel"/>
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context=".MainActivity">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{viewModel.name}"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(viewModel.age)}"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{viewModel.isMan ? @string/man : @string/woman}"/>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{v -> viewModel.log()}"
            android:text="点我"/>

    </LinearLayout>
</layout>
```

最后，改造我们的Activity代码：
```
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        // 第一种将数据填充到xml文件中的方法(代码在下面这行)，我们直接实例化了一个MainViewModel赋值给BR资源中一个叫viewModel的变量
        // binding.setVariable(BR.viewModel, MainViewModel())
        // 以下是一些说明：
        // BR就是前文提到的DataBinding资源，像R文件一样自动生成，记录所有xml中data标签内的变量名称，有点像控件id的感觉
        // viewModel来自布局文件中data标签内的variable标签中的name

        // 第二种将数据填充到xml文件中的方法(代码在下面这行),viewModel这个变量名视你在xml中variable标签中的name而定
        binding.viewModel = MainViewModel()
        // 假如你的name为user,并且class名称也为User的话(name和class的名称不一定要相同)
        // 那么代码就是binding.user = User()

        // java 代码如下
        // binding.setViewModel(new MainViewModel())
        // binding.setUser(new User())
    }

    override fun onDestroy() {
        super.onDestroy()
        // 在Activity销毁时记得解绑，以免内存泄漏
        binding.unbind()
    }
}
```

### 3， 加入DataBinding后，xml文件的一些新的用法
1. 数据的填充

    可以很明显的看到，我们在布局文件的最外层不是任何布局标签，而是layout标签。之后再引入data标签，data里面是变量集合，整个xml文件中只允许有一个data标签。data标签中可以包含多个variable。name代表变量名称，type是变量类型。在activity中，我们新建了一个binding变量，并且通过binding变量把MainViewModel实例化的对象赋值到xml文件中，这样我们在xml中就可以直接填充到对应控件中。通过@{}，我们的控件就可以直接引用到viewModel中的对应的值。就像：
    ```
    android:text="@{viewModel.name}"
    ```
2. import标签

    就像java中的import关键字一样，可以导入类型，所以我们上面的xml文件中data部份还可以这样写：
    ```
    <import type="top.cyixlq.test.MainViewModel"/>
    <variable
        name="viewModel"
        type="MainViewModel"/>
    ```
    我们还注意到，填充age属性的时候，我们是@{String.valueOf(viewModel.age)}。因为age是整数型，我们知道TextView的Text是不可以为整数型的，所以我们使用了String这个类中的方法进行了转换。按理说，String理应也需要使用import标签进行引入，然而我们并没有这么做。是的，和Java一样，java.lang包下的东西是自动引入的。
3. 三元运算符和lambda表达式以及简单运算

    我们可以看到，我们填充isMan这个属性的时候使用了三元运算符，并且使用@string/man和@string/woman作为两个可选值。看起来是不是很神奇？其实我们也可以直接这样写：
    ```
    android:text='@{viewModel.isMan ? "男" : "女"}'
    ```
    但是值得注意的是，在Windows下，我们这样写可能会报错。是关于utf-8的一个错误，具体不太清楚。如果你是java代码，在编译的时候会告诉你这个错误。如果是kotlin下，就会显示无法打印这个错误log。所以我还是推荐引用string资源。

    就像我们在xml文件中设置按钮的点击事件一样，我们可以直接引入lambda表达式，从而直接调用viewModel中的公开方法，是不是觉得简单多了？

    虽然可以直接在xml文件中进行运算了，例如字符串的拼接，数字的加减，如下所示：
    ```
    android:text="@{String.valueOf(viewModel.age + 1)}"
    android:text='"性别：" + viewModel.isMan ? "男" : "女"'
    ```
    但是，我不推荐在xml文件中进行过于复杂的运算，可以在ViewModel类中处理好之后利用函数返回。如下所示：
    ```
    <!-- xml文件中 -->
    android:text="@{viewModel.convertSex()}"

    // MainViewModel文件中
    fun convertSex(): String {
        var result = "性别："
        val sex = if (isMan) "男" else "女"
        return result + sex
    }
    ```
4. include标签的一些变化

    我们在开发中难免要进行布局的复用，这就会用到include标签了，但是如果我们引入的布局文件中也有variable怎么办，怎么才能从当前布局文件中传入到include导入的布局中呢？请直接看代码说明！我们以自己做一个标题栏为例。

    1. 首先隐藏我们原有的标题栏，在Activity的onCreate中加入下面的代码：
        ```
        supportActionBar?.hide()
        // java代码
        // ActionBar actionBar = getSupportActionBar();
        // if (actionBar != null) actionBar.hide()
        ```
    2. 新建一个layout_title_bar.xml，内容如下：
        ```
        <layout xmlns:android="http://schemas.android.com/apk/res/android">
            <data>
                <variable
                    name="title"
                    type="String"/>
            </data>
            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="@color/colorPrimary">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_centerInParent="true"
                    android:textColor="@android:color/white"
                    android:text="@{title}"/>

            </RelativeLayout>
        </layout>
        ```
    3. 在activity_main.xml中加入以下代码：
        ```
        <variable
            name="text"
            type="String" />

        <!-- 这里传过去的属性名称要与include引入的布局文件中variable的name一样 -->
        <!-- 同样，这里可以使用MainViewModel中的某个字符串属性作为值传过去，不声明一个新的variable -->
        <include layout="@layout/layout_titlt_bar"
            title="@{text}"/>
        ```
    4. 别忘了在Activity中给text赋值：
        ```
        binding.text = "测试"
        ```

### 4， 关于Activity文件中binding变量的一些说明
1. binding变量的类型是ActivityMainBinding，这个是项目build后自动生成的，根据布局文件名：activity_main.xml 来命名的类名称。也许你也发现了，它就是布局文件每个单词首字母大写，然后拼接上Binding。
2. 我们前面说过，加入DataBinding后我们可以省去一些UI相关代码，比如findviewById。那么具体是怎么操作呢。很简单，在binding变量赋值后，我们直接通过binding.控件ID就可以直接获取该控件实例。例如：binding.button.setOnClickListener(*)
3. 我们还可以将xml文件中的variable进行赋值。具体见上面Activity代码及相关注释！

### 5，数据的实时更新，双向绑定
在MainViewModel中和xml布局文件中新添加如下代码：
```
// MainViewModel中
fun oneYearLater() {
    age++
    Log.d("MyTAG", "年龄：$age")
}

<!-- 在xml布局文件中 -->
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:onClick="@{v -> viewModel.oneYearLater()}"
    android:text="一年后"/>
```
当我们点击这个按钮一年后之后会执行oneYearLater方法，里面的age属性会自增。但是，这样写好之后，我们发现age变化了，但是视图上的年龄的文字并没有刷新。我们不是说加入了DataBinding之后会自动实时刷新吗？别急，如果我们要实现实时刷新的话，我们要对MainViewModel进行小小的改造，其中有三种方法：
1. 就像下面那样，将对应属性改成这样：
    ```
    // var age = 15
    var age = ObservableInt(15)

    fun oneYearLater() {
        // age++
        val lastAge = age.get()
        age.set(lastAge + 1)
        Log.d("MyTAG", "年龄：$age")
    }
    ```
    
    将变量age声明为可观察的Int对象。其中，类似ObservableInt的变量类型还有：

    - ObservableBoolean
    - ObservableByte
    - ObservableChar
    - ObservableDouble
    - ObservableLong<br>...此处省略了一些基本数据类型
    
    对于列表和Map，还有下面这些类型：

    - ObservableList< T >
    - ObservableArrayList< T >
    - ObservableArrayMap<K,V>
    - ObservableMap<K,V>

    那么对于String或者自定义的类这种非基本数据类型，那么怎么办？DataBinding给我们提供了：ObservableField<T>，我们就可以这样用：
    ```
    val name = ObservableField<String>("张三")
    ```

    对于序列化，还有这个数据类型：
    > ObservableParcelable< T >
2. 让类继承BaseObservable：
    
    我们先新建一个ObserveViewModel的类，让它继承BaseObservable：
    ```
    class ObserveViewModel : BaseObservable() {

        private var firstName = "y"
        private var lastName = "c"

        // 这里要加上这个标签，在set方法中BR才能找到对应属性
        @Bindable
        fun getFirstName(): String {
            return firstName
        }

        @Bindable
        fun getLastName():String {
            return lastName
        }

        fun setFirstName(name:String) {
            this.firstName = name
            notifyPropertyChanged(BR.firstName)
        }

        fun setLastName(name:String) {
            this.lastName = name
            notifyPropertyChanged(BR.lastName)
        }

        // 改姓的方法
        fun changeLastName() {
            setLastName("薛")
        }

    }
    ```
    在xml布局中进行引入，并且将对应属性值进行展示以及设定按钮点击事件：
    ```
    <!-- 这段请放在data标签内 -->
    <variable
        name="observeViewModel"
        type="top.cyixlq.test.ObserveViewModel"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{observeViewModel.firstName}"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{observeViewModel.lastName}"/>
    
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="改姓"
        android:onClick="@{v -> observeViewModel.changeLastName()}"/>
    ```
    别忘了还要在Activity中进行赋值：
    ```
    val observeViewModel = ObserveViewModel()
    binding.observeViewModel = observeViewModel
    ```
3. 在Activity中进行监听。

    在ObserveViewModel类中新添加一个属性：
    ```
    val age = ObservableInt(17)
    ```

    在Activity中新加入如下代码：
    ```
    // 给按钮设置点击监听事件
    binding.btnAddAge.setOnClickListener {
        val lastAge = observeViewModel.age.get()
        observeViewModel.age.set(lastAge + 1)
    }
    // 监听ObserveViewModel中值的变化并进行回调处理
    observeViewModel.age.addOnPropertyChangedCallback(object : Observable.OnPropertyChangedCallback() {
        override fun onPropertyChanged(observable: Observable, i: Int) {
            binding.age.text = observeViewModel.age.get().toString()
        }
    })
    ```

    在布局文件中新增一个TextView展示新的属性，并添加一个按钮改变新的属性值：
    ```
    <TextView
        android:id="@+id/age"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="年龄"/>
    
    <Button
        android:id="@+id/btn_add_age"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="加一岁"/>
    ```

看过上面三种方法后，是不是觉得第一种方法最简单？是的，我个人也比较推崇第一种方法，简单粗暴，但是并不意味着其他方法就用不到了，我们还是应该根据业务需求使用不同的方法灵活变通！

了解Vue的同学知道，当我使用:value={{text}}的时候，就可以实现数据视图双向绑定。即输入框中的内容是什么，对应的属性值就是输入框中的内容。那么，DataBding也可以做到吗？答案是当然可以的。首先我们在MainViewModel中新添加一个text属性：
```
val text = ObservableField<String>("")
```
然后在activity_main.xml布局文件中多加一个EditText和一个TextView：
```
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{viewModel.text}"/>

<EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@={viewModel.text}"/>
```
这样做完之后，我们在输入框中输入什么，我们在TextView上面看到的就是什么。这样就实现了双向绑定。我们不然发现，我们实现双向绑定其实就是多加了一个“ = ”！

## OK，这一小章节我们就先到这里了，下一章节我们就介绍一下DataBinding的一些常用注解！另外本章节中出现的问题还望大家留言指正，毕竟还是在MVVM探索道路中！