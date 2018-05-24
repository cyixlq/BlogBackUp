---
title: Android自定义组件（一）—— View的绘制流程
date: 2018-05-24 08:56:30
categories: Android
tags: "Android自定义组件学习笔记"
---
### 前言
今天算是正式开始要好好研究研究安卓的自定义组件了，以前学的知识一直都很散。前端Vue，js...后端SSH，Python，PHP...五花八门！今天是时候好好整理了，毕竟最初的目标还是想学习安卓的，所以先从自定义组件开刀吧。
<!-- more -->
### 正文
 1. ##### 先说点基础的
    安卓中的所有组件全是View的子类或者间接子类，而容器ViewGroup也是View的子类。前者是单个组件，后者是组件容器。 

2. ##### Activity组成结构
    既然要说View的绘制流程，最先说的应该是能看到View的窗口说起。我们都知道，activity代表的是窗口，这里的窗口其实指的是Activity的成员变量mWindow。mWindow其实是一个PhoneWindow对象，PhoneWindow继承了Window抽象类来负责管理窗口。但是PhoneWindow并不会用来呈现界面效果，呈现界面是由PhoneWindow管理的DecorView来完成。DecorView这个类是FrameLayout的子类，也是整个View树的根。DecorView由三个部分组成，ActionBar，TitleBar和content。在SDK中platforms/android-21/data/res/layout 的目录下有一个名为 screen_title.xml 的布局文件，该布局文件是常见的窗口风格定义文件，打开后可以看到如下的定义：
    ```
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:fitsSystemWindows="true">
        <!-- Popout bar for action modes -->
        <ViewStub android:id="@+id/action_mode_bar_stub"
                android:inflatedId="@+id/action_mode_bar"
                android:layout="@layout/action_mode_bar"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:theme="?attr/actionBarTheme" />
        <FrameLayout
            android:layout_width="match_parent" 
            android:layout_height="?android:attr/windowTitleSize"
            style="?android:attr/windowTitleBackgroundStyle">
            <TextView android:id="@android:id/title" 
                style="?android:attr/windowTitleStyle"
                android:background="@null"
                android:fadingEdge="horizontal"
                android:gravity="center_vertical"
                android:layout_width="match_parent"
                android:layout_height="match_parent" />
        </FrameLayout>
        <FrameLayout android:id="@android:id/content"
            android:layout_width="match_parent" 
            android:layout_height="0dip"
            android:layout_weight="1"
            android:foregroundGravity="fill_horizontal|top"
            android:foreground="?android:attr/windowContentOverlay" />
    </LinearLayout>
    ```
    从代码中看出，ActionBar 由 ViewStub 标签定义，内容区包含了两个 FrameLayout 标签，分别代表标题栏和正文区，id 为@android:id/content 的 FrameLayout 被 inflate 成名为mContentParent的FrameLayout对象，在Activity的onCreate()方法中调用 setContentView()方法加载的布局内容终将成为 mContentParent 的子 View。

    ![组件关系图](https://upload-images.jianshu.io/upload_images/8654767-1e2b0c26bc978783.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    ##### 总结
    - Activity 类似于一个框架，负责容器生命周期及活动，窗口通过 Window 来管理；
    - Window 负责窗口管理（实际是子类 PhoneWindow），窗口的绘制和渲染交给 DecorView
    完成；
    - DecorView 是 View 树的根，开发人员为 Activity 定义的 layout 将成为 DecorView 的子
    视图 ContentParent 的子视图（有点拗口，但确实是这样）；
    - layout.xml 是开发人员定义的布局文件，最终 inflate 为 DecorView 的子组件。

    ##### 至此，我们了解了View的最基础一个绘制流程
    另外，PhoneWindow 类还关联了一个名为 mWindowManager 的 WindowManager 对象，WindowManager 会创建一个 ViewRootImpl 对象来和 WindowManagerService 进行沟通，WindowManagerService 能获取触摸事件、键盘事件或轨迹球事件，并通过 ViewRootImpl 将事件分发给各个 Actitivty；另外，ViewRootImpl 还负责 Activity 整个 GUI 的绘制。如图：

    ![Activity各组件关系图](https://upload-images.jianshu.io/upload_images/8654767-c07a850be8f2123a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. ##### View树的绘制流程
    1. 上文提到，ViewRootImpl 负责 Activity 整个 GUI 的绘制，而绘制是ViewRootImpl 的performTraversals()方法开始的，该方法使用 private 修饰，控制着 View 树的绘制流程，禁止被重写。在ViewRootImpl的源码中performMeasure()方法测量组件的大小，performLayout()方法用于子组件的定位（放在窗口的什么地方），而 performDraw()方法自然就是将组件的外观绘制出来了。

    2. performMeasure()方法负责组件自身尺寸的测量，我们知道，在 layout 布局文件中，每一个组件都必须设置 layout_width 和 layout_height 属性，属性值有三种可选模式：wrap_content、match_parent 和数值，performMeasure()方法根据设置的模式计算出组件的宽度和高度。事实上，大多数情况下模式为 match_parent 和数值的时候是不需要计算的，传过来的就是父容器自己计算好的尺寸或是一个指定的精确值，只有当模式为 wrap_content 的时候才需要根据内容进行尺寸的测量。performMeasure()方法的源码摘录如下：
        ```
        private void performMeasure(int	childWidthMeasureSpec,int childHeightMeasureSpec)	{
            try{
                mView.measure(childWidthMeasureSpec,childHeightMeasureSpec);
            }finally{
            }
        }
        ```
        对象 mView 是 View 树的根视图，代码中调用了 mView 的 measure()方法，onMeasure()方法是为组件尺寸的测量预留的功能接口，当然，也定义了默认的实现，默认实现并没有太多意义，在绝大部分情况下，onMeasure()方法必须重写。如果测量的是容器的尺寸，而容器的尺寸又依赖于子组件的大小，所以必须先测量容器中子组件的大小，不然，测量出来的宽度和高度永远为 0。

    3. performLayout()方法用于确定子组件的位置，所以，该方法只针对 ViewGroup 容器类。作为容器，必须为容器中的子 View 精确定义位置和大小。在View的layout方法中在定位之前如果需要重新测量组件的大小则要先调用 onMeasure()方法，接下来执行 setOpticalFrame()或 setFrame()方法确定自身的位置与大小，此时只是保存了相关的值，与具体的绘制无关。随后，onLayout()方法被调用，该方法是空方法。onLayout()方法在这里的作用是当前组件为容器时，负责定位容器中的子组件，这其实是一个递归的过程，如果子组件也是一个容器，该容器依然要负责他的子组件的定位，依此类推，直到所有的组件都定位完成为止。也就是说，从最顶层的 DecorView 开始定位，像多米诺骨牌一样，从上往下驱动，最后每一个组件都放到了他应该出现的位置上。onLayout()方法和上节的onMeasure()方法一样，是为开发人员预留的功能扩展接口，自定义容器时，该方法必须重写。

    4. performDraw()方法执行组件的绘制功能，组件绘制是一个十分复杂的过程，不仅仅绘制组件本身，还要绘制背景、滚动条，好消息是每个组件只需要负责自身的绘制，而且一般来说，容器组件不需要绘制，ViewGroup 已经做了大量的工作。performDraw()方法中调用了 draw()方法，draw()方法又调用了 drawSoftware()方法。绘制组件是通过 Canvas 类完成的，该类定义了若干个绘制图形的方法，通过 Paint类配置绘制参数，便能绘制出各种图案效果。为了ᨀ高绘图的性能，使用了 Surface 技术，Surface提供了一套双缓存机制，能大大加快绘图效率，而我们绘图时需要的 Canvas 对象也由是 Surface创建的。drawSoftware()方法中调用了 mView 的 draw()方法，前面说过，mView 是 Activity 界面中 View树的根（DecroView），也是一个容器（具体来说就是一个 FrameLayout 布局容器）。FrameLayout 类的 draw()方法做了两件事，一是调用父类的 draw()方法绘制自己，二是将前景位图画在了 canvas 上。 View 类的 draw()方法是组件绘制的核心方法，主要做了下面几件事：
        - 绘制背景：background.draw(canvas)
        - 绘制自己：onDraw(canvas)
        - 绘制子视图：dispatchDraw(canvas)
        - 绘制滚动条：onDrawScrollBars(canvas)   
        
        background 是一个 Drawable 对象，直接绘制在 Canvas 上，并且与组件要绘制的内容互不干扰，很多时候，这个特征能被某些场景利用。View 只是组件的抽象定义，他自己并不知道自己是什么样子，所以，View 定义了一个空方法 onDraw()。和前面讲过的 onMeasure()与 onLayout()一样，onDraw()方法同样是预留给子类扩展的功能接口，用于绘制组件自身。组件的外观由该方法来决定。dispatchDraw()方法也是一个空方法。该方法服务于容器组件，容器中的子组件必须通过 dispatchDraw()方法进行绘制，所以，View虽然没有实现该方法但他的子类 ViewGroup 实现了该方法：
        ```
        protected void dispatchDraw(Canvas canvas){
            ...
            final int count = mChildrenCount;
            final View[] children = mChildren;
            for(int i = 0;i<count;i++){
                final View child = children[i];
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE||child.getAnimation()!=null){
                    more|=drawChild(canvas,	child,	drawingTime);
                }
            }
            ...
        }
        ```
        在 dispatchDraw()方法中，循环遍历每一个子组件，并调用 drawChild()方法绘制子组件，而子组件又调用 View 的 draw()方法绘制自己。组件的绘制也是一个递归的过程，说到底 Activity 的 UI 界面的根一定是容器，根容器绘制结束后开始绘制子组件，子组件如果是容器继续往下递归绘制，否则将子组件绘制出来……直到所有的组件正确绘制为止。

    ##### 总结
    UI 界面的绘制从开始到结束要经历几个过程：
    - 测量大小，回调 onMeasure()方法
    - 组件定位，回调 onLayout()方法
    - 组件绘制，回调 onDraw()方法

### 至此，我们的第一章就算是结束了，博主也没太看懂书中的内容，所以这片博文还是会觉得比较乱，之后还要反复阅读揣摩，加上自己的总结来重新写一下这篇博文。

