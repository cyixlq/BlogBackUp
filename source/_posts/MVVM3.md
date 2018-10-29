---
title: Android MVVM探索(三) - ViewModel，DataBinding，LiveData混合三打
date: 2018-10-29 09:07:44
tags: [MVVM]
categories: Android
---
### 之前几小章我们讲了DataBinding，其中将一个普通类化身为ViewModel，但是以我的观点来看，他仅仅只是一个普通类，一个将各种可观察属性封装起来的普通类，而这个普通类我们还在里面定义了各种相应按钮点击事件等方法，其实这些都违背了官方的建议的，我只是想让大家知道可以这样做而已。所以我们要介绍Android Jetpack中正统的ViewModel类，以及一些它的最佳实践指南。

[本篇文章代码地址](https://github.com/cyixlq/MVVMTest)

[官方中文教学视频地址](https://www.bilibili.com/video/av29949898)

<!-- more -->

## Android MVVM探索系列
[Android MVVM探索(一) - DataBiding初解](https://cyixlq.top/2018/10/22/MVVM1/)

[Android MVVM探索(二) - DataBiding常用注解](https://cyixlq.top/2018/10/23/MVVM2/)

[Android MVVM探索(三) - ViewModel，DataBinding，LiveData混合三打](https://cyixlq.top/2018/10/29/MVVM3/)

### Android Jetpack是谷歌为了帮助开发者们更快更高效地开发安卓应用而推出来的一套组件。Android Jetpack包含了开发库，工具以及最佳实践指南。而我们今天要讲的ViewModel类是属于Android Jetpack库中的lifecycle库。说到这，顺带解释以下。lifecycle，中文意思为生命周期。所以这个库的存在就跟它的中文含义一样，它可以有效避免内存泄漏和解决Android常见的生命周期难题。（如果各位看官不知道内存泄露的可以去好好补补课）lifecycle最近发布了2.0版本，在这个版本中，可以结合DataBinding进行使用，那可以说是方便太多了。

#### 一，ViewModel的定义。
来自官方的解释：ViewModel类是用来保存UI数据的类，它会在配置变更（即 Configuration Change，例如手机屏幕的旋转）之后继续存在。

#### 二，它的一些特点。
我们都知道，当手机屏幕发生旋转的时候，Activity会被重新创建，也就是说生命周期又将从onCreate开始，如果你此时不及时保存，那么一些UI数据将会丢失，这样肯定是会出问题的。但是，ViewModel并不会受此影响，即便手机屏幕发生旋转，ViewModel依然存在，这样的话Activity的UI数据便可以保存下来。

#### 三，最佳实践（官方推荐做法）。
1. 所有Activity的UI相关数据应该保存在ViewModel中，而不是保存在Activity中。这样做的好处是，在配置变更的时候，你应用的UI数据仍然存在。即使ViewModel这么强大，但它也不应该不承担过多责任，当有UI数据处理等相关事件建议创建Presenter类，或者创建一个更成熟的架构。
2. Activity负责展示UI数据，并接收互动（一般来说是与用户的互动）。但是Activity不应当处理这些互动。
3. 在应用需要加载数据或者保存数据的时候，建议创建一个Repository的存储区类，里面放置存储与加载应用数据的API。
4. ViewModel不应持有Context，就像之前说的：
    > 它会在配置变更（即 Configuration Change，例如手机屏幕的旋转）之后继续存在。
    
    所以，ViewModel生命周期远比Activity，Fragment等生命周期更长，具体如下图所示。如果你这样做了，加入在屏幕旋转情况下，原Activity将会销毁，新的Activity将会被创建。而ViewModel会一直持有原Activity，这样便会造成内存泄漏。如果你的ViewModel确实需要Context，那么你的ViewModel可以继承AndroidViewModel，这样你的ViewModel中会有Application的引用。<br>
    ![QQ截图20181029142003.png](https://upload-images.jianshu.io/upload_images/8654767-3367001936c06aac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
5. ViewModel不应当取代onSaveInstanceState方法。尽管ViewModel很出色了，但是它和onSaveInstanceState依然是相辅相成的作用。因为，当进程被关闭时，ViewModel将会被销毁，但是onSaveInstanceState不会受到影响。（个人猜想：比如在后台内存紧张情况下，你的应用处于后台被系统释放了，ViewModel会被销毁，但是你通过onSaveInstanceState存储下来的数据在你的应用重新回到前台时仍然可以被恢复）
    ![QQ截图20181029143111.png](https://upload-images.jianshu.io/upload_images/8654767-0a4131db00ef8d7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6. ViewModel与Activity生命周期对比图：
    ![viewmodel-lifecycle.png](https://upload-images.jianshu.io/upload_images/8654767-b7cb23337850512e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 四，ViewModel的用法
1. 引入lifecycle库：
    ```
    implementation "android.arch.lifecycle:extensions:1.1.0"
    annotationProcessor "android.arch.lifecycle:compiler:1.1.0"
    ```
2. 首先创建出我们的ViewModel类。我们需要新建一个普通类，（虽然类名是随意的，但是作为一名合格的程序员，我们取得每一个名字要具有规范性，要让代码阅读者一看名字就知道这个类或者这个变量是干嘛的）让它继承ViewModel类，并且在其中存放UI相关数据：
    ````
    // 假设我们要存放的UI数据就是User对象
    // 先新建一个实体对象
    data class User(val name: String, val age: Int, val sex: Int)

    // 新建ViewModel类
    // 新建ViewModel类
    class UserViewModel : ViewModel() {
        val user = User("张三", 21, 1)
    }
    ````

3. 新建Activity，并且加入实例化ViewModel的代码。这里我们实例化ViewModel不再是new一下：
    ```
    class ViewModelActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_view_model)
            val userViewModel = ViewModelProviders.of(this).get(UserViewModel::class.java)
        }
    }
    ```
    至此，我们一个ViewModel就创建好了。

4. 感觉单单一个ViewModel的存在没有很大的价值，但是如果搭配上LiveData和DataBinding你就能体会到什么是飞一样的感觉。用上这些，你就能够创建反应式界面（最基本的只要LiveData和ViewModel就可以创建反应式界面。或者单单DataBinding就可以完成，但是不具备生命周期感知能力，我们需要手动处理生命周期问题）。也就是说，当你底层数据发生变动时（这里暂时只是ViewModel中数据发生变动），UI会自动刷新。
5. ViewModel只提供一个默认的无参构造函数，如果你需要一个有参构造函数，那么就需要使用ViewModelFactory这个类，具体使用方法如下所示(摘自官方sunflower Demo代码)：
    ```
    // 新建一个Factory类，用来提供带有有参构造函数的ViewModel实例，必须继承ViewModelProvider.NewInstanceFactory，然后重写create方法
    class MessageViewModelFactory(private val message: Message) : ViewModelProvider.NewInstanceFactory() {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel?> create(modelClass: Class<T>): T {
            return MessageViewModel(message) as T
        }
    }

    // 新建一个带有有参构造函数的ViewModel
    class MessageViewModel(val message: Message) : ViewModel()

    // Activity中初始化ViewModel的代码也要进行改动
    // 创建Factory对象
    val factory = MessageViewModelFactory(Message("我是通过有参构造函数直接初始化的信息内容",
            "我是通过有参构造函数直接构造出来的信息发送人"))
    // 通过Factory对象初始化带参构造函数的ViewModel
    val messageViewModel = ViewModelProviders.of(this, factory).get(MessageViewModel::class.java)
    // 将MessageViewModel实例赋值给xml中的msg
    binding.msg = messageViewModel
    ```
    这样便可以完成带有有参构造函数的ViewModel的初始化。

#### 五，搭配上LiveData

1. LiveData简介：LiveData是一种具有生命周期感知能力的可观察数据持有类。它同属于Android Jectpack中的lifecycle库。LiveData对象通常保存在我们上面讲的ViewModel中。
2. 结合ViewModel的使用。我们说过，使用LiveData加ViewModel可以创建一个反应式界面，那么我们应该怎么做呢？我们需要改写上面的UserViewModel，让其中的user具有可观察性：
    ```
    class UserViewModel : ViewModel() {
        val user = MutableLiveData<User>()
    }
    ```

3. 接着，我们在ViewModelActivity中监听UserViewModel中的user属性的变化，并且在它变化后进行相应的操作：
    ```
    <!-- 为了演示效果，我们在布局文件activity_view_model中添加了一些控件 -->
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context=".viewmodel.ViewModelActivity">

        <TextView
            android:id="@+id/tv_vm"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <Button
            android:id="@+id/bt_vm"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="设置姓名"/>
    </LinearLayout>

    // 在Activity中设置数据变化监听，并且进行相应处理
    class ViewModelActivity : AppCompatActivity() {
        private lateinit var tv: TextView
        private lateinit var bt: Button

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_view_model)
            tv = findViewById(R.id.tv_vm)
            bt = findViewById(R.id.bt_vm)
            val userViewModel = ViewModelProviders.of(this).get(UserViewModel::class.java)
            // 监听ViewModel中user的变化，当它变化时，将TextView重新设置文字
            userViewModel.user.observe(this, Observer {
                tv.text = it?.name
            })
            // 为按钮设置点击事件，点击后设置user的值
            bt.setOnClickListener{
                val user = User("张三", 21, 1)
                userViewModel.user.value = user
                // Java代码
                // userViewModel.user.setValue(user)
            }
        }
    }
    ```
    写完以上代码，当我们点击按钮的时候TextView就会显示“张三”二字了。
4. 上面代码中除了setValue，还有一个方法叫postValue。区别在于setValue只可以在主线程执行（即UI线程），postValue只可以在后台线程运行。
5. LiveData能够感知生命周期的好处：1，当Activity不在屏幕上时（不可见），LiveData不会出发没必要的界面更新；2，当Activity被销毁时，LiveData将自动清空与Observer的连接；

#### 六，搭配上DataBinding
我们看到，上面的代码还是有些繁琐，我们还要自己写代码监听数据变化，并且自己手动去更新UI。之前我们DataBinding可不是这样的。是的，我们还可以更简单。别忘了，lifecycle2.0是支持了DataBinding数据绑定的。我们可以通过以下步骤来结合DataBinding使用：

1. 像DataBinding中那样写布局，所以将activity_view_model改成如下所示：
    ```
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools">
        <data>
            <variable
                name="viewModel"
                type="top.cyixlq.test.viewmodel.UserViewModel"/>
        </data>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            tools:context=".viewmodel.ViewModelActivity">

            <TextView
                android:id="@+id/tv_vm"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{viewModel.user.name}"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{String.valueOf(viewModel.user.age)}"/>

            <Button
                android:id="@+id/bt_vm"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="设置姓名"/>

        </LinearLayout>
    </layout>
    ```

2. 注释掉ViewModelActivity中的之前的代码，并重新写，所以有用的代码就是下面这样：
    ```
    class ViewModelActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_view_model)
            val userViewModel = ViewModelProviders.of(this).get(UserViewModel::class.java)
            val binding = DataBindingUtil.setContentView<ActivityViewModelBinding>(this, R.layout.activity_view_model)
            // Java代码
            // ActivityViewModelBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_view_model)

            binding.viewModel = userViewModel
            // java代码
            // binding.setViewModel(userViewModel)

            // 让xml内绑定的LiveData和Observer建立连接，也正是因为这段代码，让LiveData能感知Activity的生命周期
            binding.setLifecycleOwner(this)
        }
    }
    ```

3. 为了验证效果，我们设置一下按钮的的点击事件，当按钮点击后TextView显示姓名年龄的信息：
    ```
    bt_vm.setOnClickListener { 
        userViewModel.user.value = User("李四", 22, 1)
    }
    // java代码
    // findViewById(R.id.bt_vm).setOnClickListener(new View.OnClickListener() {
    //     userViewModel.user.setValue(new User("李四", 22, 1))
    // })
    ```

经过以上的步骤就可以结合DataBinding使用了