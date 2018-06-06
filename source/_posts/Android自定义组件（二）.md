---
title: Android自定义组件（二）—— Graphics2D API
date: 2018-05-24 10:53:45
categories: Android
tags: "Android自定义组件学习笔记"
---
### 前言
本章将向大家介绍 Graphics2D 涉及到的常用类，如基本的数据结构类、位图类、绘图类等，并介绍和使用常见的绘图方法。

<!-- more -->

### 正文

- #### Point 类和 PointF 类	
    1. Point类代表坐标中的一个点。它实现了Parcelable 接口，支持序列化与反序列化。Point 类定义了两个 int 成员 x 和 y，代表点的 x 坐标和 y 坐标。众所周知，安卓中的坐标和数学中的坐标是有些出入的。安卓中，左上角屏幕的点为原点，向右为x正轴，向下为y正轴，而在数学中此时的向下y是为反轴的，也就是数值是负数的，安卓中却是正数的。
    2. 我们先来看看Point的构造函数：
        ```
        public Point() {}
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
        public Point(Point src) {
            this.x = src.x;
            this.y = src.y;
        }
        ```
        我们可以看到只是提供了一些设置坐标初始值的构造方法，同时也可以将一个Point的坐标赋值给另一个Point。
    3. Point类还提供了三个方法改变坐标值：
        ```
        public void set(int x, int y)	{
            this.x = x;
            this.y = y;
        }
        public final void negate() {
            x = -x;
            y = -y;
        }
        public final void offset(int dx, int dy) {
            x += dx;
            y += dy;
        }
        ```
        我们可以看到set方法简单的将x，y的值传入然后进行赋值，而negate只是取反，offset则是设置偏移量，正负符号决定坐标偏移的方向。
    4. PointF 类和 Point 类的定义是完全一样的，最大的不同就是成员变量 x、y 的类型不是 int 而是 float，这也是加了后缀“F”的原因。不过，PointF 提供了一个很贴心的功能，定义了 length()方法计算坐标原点（0，0）到（x，y）之间的距离，而且有两个版本：静态的 length()和非静态的length()，代码如下：
        ```
        public final float length() {	
            return length(x,y);	
        }
        public static float length(float x, float y) {
            return FloatMath.sqrt(x * x + y * y);
        }
        ```
        可以看出，使用了数学中的勾股定理求得第三边长度。

- #### Rect 类和 RectF 类
    1. Rect类定义了一个矩形结构，用来画矩形的，它同样实现了Parcelable 序列化接口。Rect 类定义了 left、top、right、bottom 四个成员变量，我们需要正确理解这 4 个成员变量的作用：
        - left：矩形左边线条离 y 轴的距离
        - top：矩形上面线条离 x 轴的距离
        - right：矩形右边线条离 y 轴的距离
        - bottom：矩形底部线条离 x 轴的距离 
        
        我们看图理解：

        ![left、top、right、bottom变量含义图解](https://upload-images.jianshu.io/upload_images/8654767-7988c20fc961ed7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    2. 矩形是一种非常常见的图形结构，并且能衍生出更多的图形，如椭圆、扇形、弧线等等；矩形还能进行各种图形运算，如交集、并集等等，所以，与之对应的 Rect 类功能也更加复杂。有人会疑惑为什么不直接指定左上角的坐标、宽度和高度来确定一个矩形，因为指定 top、left、right 和 bottom 更符合坐标系的数学逻辑，也能更好的支持矩形的计算。
    3. Rect 的主要功能有：
        - 初始化：主要有两种初始化的方法：一是直接指定 left、top、right、bottom 等4 个成员变量的值，二是从另一个 Rect 对象中复制。下面是 Rect 的三个构造方法（简单展示）：
            ```
            Rect()
            Rect(int left, int top, int right, int bottom)
            Rect(Rect r)
            ```
            同样，这几个构造函数一般用来设置初始值，一目了然。最后一个支持将一个Rect对象的参数赋值给另一个Rect对象。

        - 增值计算：根据 left、top、right、bottom 等 4 个成员变量计算矩形的宽度、高度或中心点的坐标，主要的方法定义如下：
            ```
            //判断 Rect 是否为空，也就是矩形区域面积是否为 0 或者为无效矩形。
            public final boolean isEmpty() {
                return left >= right || top >= bottom;
            }

            //返回矩形的宽
            public final int width() {
                return right - left;
            }

            //返回矩形的高
            public final int height() {
                return bottom - top;
            }

            //计算矩形中心点的 x 坐标，右移一位相当于除以 2，移位运算比普通的除法运算效率更高。
            public final int centerX() {
                return (left + right) >> 1;
            }

            //计算矩形中心点的 y 坐标，右移一位相当于除以 2，移位运算比普通的除法运算效率更高。
            public final int centerY() {
                return (top + bottom) >> 1;
            }

            //计算矩形中心点的 x 坐标，返回 float 类型，结果更精确。
            public final float exactCenterX() {
                return (left + right) * 0.5f;
            }

            //计算矩形中心点的 y 坐标，返回 float 类型，结果更精确。
            public final float exactCenterY() {
                return (top + bottom) * 0.5f;
            }
            ```
        - 改变矩形的位置或大小，通过修改 left、top、right 和 bottom 等 4 个成员变量的值，获取矩形位置平移、放大、缩小等结果:
            ```
            //将矩形的 left、top、right 和 bottom 置 0。
            public void setEmpty() {
                left = right = top = bottom = 0;
            }
            
            //给 left、top、right 和 bottom 重新赋值。
            public void set(int left, int top, int right, int bottom) {
                this.left = left;
                this.top = top;
                this.right = right;
                this.bottom = bottom;
            }
            
            //矩形的 left、top、right 和 bottom 来自于另一个矩形 src。
            public void set(Rect src) {
                this.left = src.left;
                this.top = src.top;
                this.right = src.right;
                this.bottom = src.bottom;
            }
            
            //矩形的 left 和 right 同时移动相同的距离 dx，矩形的 top 和 bottom 同时移动相同的距离 dy，实际上就是将矩形移动（dx、dy）距离，正负决定移动的方向。
            public void offset(int dx, int dy) {
                left += dx;
                top += dy;
                right += dx;
                bottom += dy;
            }

            //offsetTo()方法也是移位，和 offset()不同的是前者是绝对定位，后者是相对定位。
            public void offsetTo(int newLeft, int newTop) {
                right += newLeft - left;
                bottom += newTop - top;
                left = newLeft;
                top = newTop;
            }

            //实现了矩形的缩放功能，缩放中心点就是矩形的中心点，要注意的是 dx、dy 为正数时表示缩小，负数表示放大。
            public void inset(int dx, int dy) {
                left += dx;
                top += dy;
                right -= dx;
                bottom -= dy;
            }
            ```
        - 包含测试：支持一个点是否位于矩形内和一个矩形是否位于另一个矩形内:
            ```
            // 判断点（x，y）是否位于矩形内。
            public boolean contains(int x, int y)	{
                return left < right && top < bottom && x >= left && x < right && y >= top && y < bottom;
            }
           
            //判断传递过来的矩形是否位于矩形内。
            public boolean contains(int left, int top, int right, int bottom) {
                return this.left < this.right && this.top < this.bottom && this.left <= left && this.top <= top && this.right >= right && this.bottom >= bottom;
            }

            //判断传递过来的矩形是否位于矩形内。
            public boolean contains(Rect r) {
                return this.left < this.right && this.top < this.bottom && left <= r.left && top <= r.top && right >= r.right && bottom >= r.bottom;
            }
            ```

#### 好的，今天我们就先写到这，明天继续咯！