---
title: 一个小巧的日期选择器
date: 2019-04-24 16:16:55
tags: [自定义View实战]
categories: Android
---
### 前言
不知道你和我是否有一样有过这样的想法。身在外包工作，很多时候，只是一个小小的需求，却要写一个差不多精致的UI，但是这个UI耗费的时间却需要很久。但是对于你来说，只想快速完成需求，UI只是一个吃力不讨好的事情，为此你疲于应对，然后去网上找别人写好的，但是却偏偏没有你想要的。更有甚至为了一个小小的UI引入一个体积很大的依赖库。

是的，我有过这样的想法，前不久我接到了一个新功能开发，里面有一个时间选择弹框，可以选择开始时间和结束时间，效果就相当于下图这样：
![效果图](/images/一个小巧的日期选择器效果图.gif)

<!-- more -->

- 效果文字描述：
    - 可以选择开始那天和结束那天不是同一天
    - 也可以单选一天，即开始时间和结束时间是同一天
    - 点击已经点选的开始或者结束那天可以取消选择，然后下一次点击又可以变成一个新的时间段

一开始我并不想做这么个控件，毕竟时间有限，一直在赶需求，只是简单地做了两个系统自身的DatePickerDialog弹出来，一个是让对方选择开始时间，选择后消失弹出第二个选择结束时间。后来对方不满意，在其它需求做完了的情况下开始啃这个了。于是乎，在今天有空的情况下，我单独将这个控件提取出来，并且做了一定的封装和可自定义。于是就有了现在这篇文章。
### 项目地址
[https://github.com/cyixlq/Cwidget](https://github.com/cyixlq/Cwidget)
### 使用方法
直接使用如图中样式的话：
```java
DatePickerDialogFragment mDatePickerDialog = new DatePickerDialogFragment();
final SelectRule rule = new SelectRule();
rule.setCanSelectAfterToday(false); // 设置是否可以选择超过今天的时间
rule.setEnable(true); // 设置是否开启时间选择功能
rule.setMaxDayCount(30); // 设置最多可以选择多少天
mDatePickerDialog.setSelectRule(rule); // 如果不设置rule默认不能超过今天，开启时间选择，最多天数不限制
mDatePickerDialog.setOnTimeSelectListener((start, end) -> {
    final String msg = "开始时间：" + start.toString() + "\n" + "结束时间：" + end.toString();
        Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
    });
mDatePickerDialog.show(getSupportFragmentManager());
```
如果要自定义Item的布局样式，请参照[SimpleCalendarAdapter](https://github.com/cyixlq/Cwidget/blob/master/library/src/main/java/top/cyixlq/widget/calendar/SimpleCalendarAdapter.java)

### THE END