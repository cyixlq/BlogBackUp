---
title: Android自定义View基础（一）
date: 2018-03-14 08:56:29
categories: Android
tags: ["自定义View基础系列"]
---
安卓就业市场现在饱和度比较高，个人感觉很难混，所以我认为解决的办法就是不断提高自己的技术。当然，如果撑不下去就只能转行了！  
话不多说，开始动手吧，让我们先来了解一下自定义view的一些基础知识。

View的基础知识：
---
1. 我们先重写他的四个构造方法，然后我们就会发现，四个参数的构造方法出现报错，**Call requires API level 21 (current min is 19): new android.view.View**，大致意思是api等级必须为21才会用的构造方法，而我现在的最低api等级为19，那我们就删除吧。所以我现在的代码如下：
<!-- more -->
```
public class TestView extends View {
    
    public final String TAG="TestView";
    
    public TestView(Context context) {
        super(context);
        Log.d(TAG,"执行了一个参数的构造方法！");
    }

    public TestView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        Log.d(TAG,"执行了两个参数的构造方法！");
    }

    public TestView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        Log.d(TAG,"执行了三个参数的构造方法！");
    }
    
}
```
布局文件代码如下：
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:cy="http://schemas.android.com/apk/res-auto"
    tools:context="com.cy.diyview.MainActivity">

    <com.cy.diyview.widght.TestView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</RelativeLayout>
```

我们重写三个构造函数，并且打印log，看看三个构造函数的执行顺序。好的，编译运行。log打印结果如下：
```
03-21 09:33:35.799 2095-2095/com.cy.diyview D/TestView: 执行了两个参数的构造方法！
```
我们可以看到它只执行了两个参数的构造函数。然后我们将界面布局代码添加的自定义控件删除，在Activity中添加如下代码，让它在Java代码中实例化：  

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        TestView testView=new TestView(this);
    }
}
```
打印出来的log如下所示：
```
03-21 09:37:29.339 2297-2297/com.cy.diyview D/TestView: 执行了一个参数的构造方法！
```
以上，我们得出一个结论：
- Java代码直接new一个TestView实例的时候，会调用这个只有一个参数的构造函数；
- 在默认的XML布局文件中创建的时候调用这个有两个参数的构造函数。AttributeSet类型的参数负责把XML布局文件中所自定义的属性通过AttributeSet带入到View内；  
- 构造函数中第三个参数是默认的Style，这里的默认的Style是指它在当前Application或者Activity所用的Theme中的默认Style，且只有在明确调用的时候才会调用
- 四个参数的构造函数是在API21的时候才添加上的。  

我们继续修改自定义view的代码，添加如下代码：
```
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    Log.d(TAG, "onSizeChanged：w=" + w + ",h=" + h + ",oldw=" + oldw + ",oldh=" + oldh);
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Log.d(TAG, "onDraw");
}
```
然后将Activity中的实例化view的代码删除，在布局文件中加上我们的自定义view，继续编译运行一下，打印的log如下：
```
03-21 09:43:33.826 2499-2499/? D/TestView: 执行了两个参数的构造方法！
03-21 09:43:33.914 2499-2499/? D/TestView: onSizeChanged：w=720,h=1173,oldw=0,oldh=0
03-21 09:43:33.942 2499-2499/? D/TestView: onDraw
```
这回我们弄清楚了view运行的一个工作流程，先是执行构造方法，然后是onSizeChanged，然后是onDraw绘制。    
**在自定义View中，我们通常只重写前三个构造函数，第四个四个参数的构造函数不重写。**  但是我们发现它不会调用第三个构造函数啊，所以我们一般让所有的构造方法调用三个参数的构造方法，然后在三个参数的构造方法中获得自定义属性。   
我们需要注意的是super应该改成this，然后让一个参数的构造方法引用两个参数的构造方法，两个参数的构造方法引用三个参数的构造方法，代码如下：
```
public TestView(Context context) {
    this(context,null);
    Log.d(TAG,"执行了一个参数的构造方法！");
}

public TestView(Context context, @Nullable AttributeSet attrs) {
    this(context, attrs,0);
    Log.d(TAG,"执行了两个参数的构造方法！");
}

public TestView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    Log.d(TAG,"执行了三个参数的构造方法！");
}
```
编译运行，打印的log如下：
```
03-21 09:55:41.041 2702-2702/com.cy.diyview D/TestView: 执行了三个参数的构造方法！
03-21 09:55:41.041 2702-2702/com.cy.diyview D/TestView: 执行了两个参数的构造方法！
03-21 09:55:41.114 2702-2702/com.cy.diyview D/TestView: onSizeChanged：w=720,h=1173,oldw=0,oldh=0
03-21 09:55:41.133 2702-2702/com.cy.diyview D/TestView: onDraw
```


至此，一些View的基础流程我们就写到这。
---
