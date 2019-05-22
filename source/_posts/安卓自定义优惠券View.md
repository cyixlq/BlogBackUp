---
title: 安卓自定义优惠券View
date: 2018-03-14 08:56:29
categories: Android
tags: [自定义View实战]
---

### 欢迎来我的个人博客查看更多文章: [Cy的个人博客](https://cyixlq.top/)

不久之前写过一篇基础的自定义View的博文，今天就来实践一下自定义View，参考了网上一个自定义View控件的博文完成的（毕竟还是小白）。以下是我完成的一些步骤：  
1. 首先肯定需要编写一个类，让它继承自View（感觉我这个简单的自定义view继承RelativeLayout就够了），然后重写构造方法咯，这个简单，代码如下（一般大家都重写三个构造方法，虽然不太理解，但是对于我这个简单的自定义View是重写两个构造方法是能实现的）：
```
public class QuanView extends RelativeLayout{

    public QuanView(Context context) {
        super(context);
    }

    public QuanView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

}
```
2. 然后就是初始化画笔，然后设定一些属性咯，所以现在的代码如下：
```
public class QuanView extends RelativeLayout{

    private Paint mPaint;

    private float mGap=20;    //圆和圆之间的间距
    private float mRadius=15; //圆的半径
    private float mRemain;    //画完圆和圆间距后多出来的距离
    private int mCircleNum;   //圆圈的数量

    public QuanView(Context context) {
        super(context);
        initPaint();
    }

    public QuanView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initPaint();
    }

    private void initPaint(){
        mPaint=new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.WHITE);
    }

}
```
3. 在上一篇自定义博文中我们知道onSizeChanged方法会在onDraw方法前执行，我们可以在这个方法中计算好需要绘制的圆圈的个数，所以onSizeChanged方法中代码如下：
```
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (mRemain == 0) {
            //计算不整除的剩余部分
            mRemain = (int) (w - mGap) % (2 * mRadius + mGap);
        }
        mCircleNum = (int) ((w - mGap) / (2 * mRadius + mGap));
    }
```
4. 接下来就是最重要的部分了，重写onDraw方法，绘制自定义View，onDraw代码如下：
```
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        for (int i = 0; i < mCircleNum; i++) {
            //计算出要画的圆圈的x轴位置
            float x = mGap + mRadius + mRemain / 2 + ((mGap + mRadius * 2) * i);
            //从x位置开始画上面的圆圈
            canvas.drawCircle(x, 0, mRadius, mPaint);
            //从x位置开始画下面的圆圈
            canvas.drawCircle(x, getHeight(), mRadius, mPaint);
        }
    }
```
5. 至此，自定义的优惠券View已经可以使用了，让我们把它加入到布局文件中，代码如下 ：
```
    <top.cyixlq.view.widght.QuanView
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:background="@color/colorPrimary">
    </top.cyixlq.view.widght.QuanView>
```
6. 运行后的结果如图所示：
![20180327121717.png](https://user-gold-cdn.xitu.io/2018/10/23/1669fb8750dfa307?w=518&h=887&f=png&s=19545)
7. 添加自定义属性，我们虽然能正常使用了，但是各个用户间需求不同我们还要能够自定义啊，这样才能用的舒心啊！首先就要在values目录下新建一个文件了，attrs.xml，然后在文件中加入以下代码：
```
<?xml version="1.0" encoding="utf-8" ?>
<resources>
    <declare-styleable name="QuanView">
        <!--圆半径大小-->
        <attr name="radius" format="dimension" />
        <!--圆与圆之间的间隔-->
        <attr name="gap" format="dimension" />
    </declare-styleable>
</resources>
```
8.  修改View的构造函数，读取布局文件中传来的参数：
```
public QuanView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initPaint();
        TypedArray typedArray=context.obtainStyledAttributes(attrs, R.styleable.QuanView);
        mGap = typedArray.getDimensionPixelOffset(R.styleable.QuanView_gap,20);//最后那个参数代表默认值，如果布局文件中没有传入该参数则使用默认值
        mRadius = typedArray.getDimensionPixelOffset(R.styleable.QuanView_radius,15);
        typedArray.recycle();
}
```
9. 在布局文件中加上我们的自定义的参数，代码如下：
```
<top.cyixlq.view.widght.QuanView
        android:layout_width="match_parent"
        android:layout_height="150dp"
        app:gap="2dp"
        app:radius="2dp"
        android:background="@color/colorPrimary">
</top.cyixlq.view.widght.QuanView>
```
（别忘了，使用了自定义的参数的话要引入一个命名空间哦，如下所示）
```
xmlns:app="http://schemas.android.com/apk/res-auto"
```
10. 如果觉得这样不够帅的话，app这个字段是可以换的，例如:
```
xmlns:cy="http://schemas.android.com/apk/res-auto"
```
这时候在布局文件中的自定义组件的那一段代码报错了
![命名空间报错了](https://user-gold-cdn.xitu.io/2018/10/23/1669fb8750d59579?w=671&h=235&f=png&s=27555)
这时候仅仅需要将布局中自定义的代码换成如下所示就可以了。（将app换成引入时候的名称cy）
```
<top.cyixlq.view.widght.QuanView
        android:layout_width="match_parent"
        android:layout_height="150dp"
        cy:gap="2dp"
        cy:radius="2dp"
        android:background="@color/colorPrimary">
</top.cyixlq.view.widght.QuanView>
```
这样是不是就具有标志性了呢？是不是更帅一点了呢？
最后我们再来看看修改后的效果吧！
![我们会发现锯齿变小了好多](https://user-gold-cdn.xitu.io/2018/10/23/1669fb875159a0ac?w=439&h=744&f=png&s=17843)

#### 至此，我们一个简单的自定义View就到此为止啦!拜了个拜。。。