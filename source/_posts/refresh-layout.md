---
title: 自定义一个下拉刷新控件
date: 2019-02-19 10:01:46
tags: [自定义View实战]
categories: Android
---
第一次尝试写一个下拉刷新控件，一开始的目的只是想了解dispatchTouchEvent，onInterceptTouchEvent和onTouchEvent这几个事件的分别，没想到最后竟然写了一个刷新控件。好的，废话不多说，先来看看效果图：
{% qnimg 自定义下拉刷新效果图.gif title:自定义下拉刷新效果图.gif alt:自定义下拉刷新效果图.gif %}`

<!-- more -->

首先，我们还是需要搞清楚我上面说的这三个事件：
1. 先来看dispatchTouchEvent，这个事件是用来分发touch事件的，从Activity中的窗口开始进行事件分发，首先到根View，再到根View的子View依次传递。在我的测试中，无论是返回true还是false，都无法将touch事件分发到下一级，也就是子View。这个问题也困扰了我半天。看了别人自定义的下拉刷新控件，好像都没有重写这个方法。因此，个人的做法还是将此事件交给父类处理。对于这个事件我觉得我个人还得多了解了解！
2. 其次看看onInterceptTouchEvent事件，这个方法就是拦截touch事件，你可以根据touch的事件类型分别进行拦截，本例中下拉刷新控件就需要利用此特性。返回false代表不拦截，返回true代表拦截。
3. onTouchEvent事件的作用就是处理touch事件，如果此View中没有下一级并且上一级没有对touch事件进行拦截或者此View中对touch事件进行了拦截，touch事件最终就会在此事件中处理，如果不处理的话就返回false，就会返回到上一级处理。如果处理了则返回true，这样的话就不会返回到上一级处理。
##### 开始我们的自定义下拉刷新View
我们先来理一理下拉刷新的逻辑。首先如果用户手指向上滑动，我们不需要进行事件的拦截，交给子View处理。如果用户手指是向下滑动的时候就要进行处理了。首先要看看刷新控件中的子View能不能继续向上滚动，也就是说子View有没有滚动到顶，如果到顶了，用户继续向下滑动的话就开始显示头部的刷新视图。至于是到顶后，拿开手指后再下拉显示刷新视图还是到顶后直接继续下拉就可以显示刷新视图就看项目需要了。我是实现的前者。然后就是显示刷新视图后，用户下拉多少，顶部就有多少留白，然后提示文字始终在留白的最中间位置。下拉到一定位置提示松手开始刷新。当下拉到最大距离，留白不再加大。最后松手，留白减少，并且提示正在刷新，最后提示刷新结果。

首先我定义了一些变量来记录一些需要用到的值，变量说明都在注释中：
```
// 每次触摸事件中第一次接触屏幕的Y坐标
private float downY;
// 手指在Y轴的滑动距离
private float dY;
// 在刷新布局中的子View
private View mTarget;
// 最大Y轴滑动距离
private float maxDY = 300;
// 头部View
private View headerView;
// 下拉开始时显示的文字
private String readyText = "下拉开始刷新";
// 下拉到触发刷新的下拉距离之后的提示文字
private String refreshOkText = "松开开始刷新";
// 正在刷新时候提醒的文字
private String refreshingText = "正在刷新";
// 刷新成功的提示文字
private String refreshSuc = "刷新成功！";
// 刷新失败提醒的文字
private String refreshFail = "刷新失败！";
// 触发刷新的距离
private float refreshDist = maxDY / 2;
// 滑动多少距离才算是滑动，否则有时候是点击也会误触发滑动
private int minDist;
// 是否触发了刷新
private boolean canRefresh = false;
// 正在刷新？
private boolean isRefreshing = false;
// 控件状态监听
private RefreshStateListender listener;
// 状态表示代码
private final int READY_REFRESH = 0; // 刚开始下拉时候的状态
private final int CAN_REFRESH = 1; // 已经可以触发刷新的状态
private final int ON_REFRESH = 2; // 正在刷新的状态
private final int ON_FINISH = 3; // 刷新完成的状态
```
接着我们就要来处理一下事件的拦截，从我们上面理好的逻辑中知道，我们主要处理用户下拉手势。首先我们就应当知道用户到底是在上拉还是在下拉。我的做法是，当用户第一次触摸到屏幕的时候，我记录下这个点的Y轴位置为初始Y轴位置，然后在用户的滑动过程中，获取滑动的点的Y轴位置减去初始Y轴位置。如果结果为负数，代表是上拉，如果是正数就代表下拉然后对事件进行拦截。但是，我在实现过程中发现不能通过判断正负拦截，因为点击也是属于touch事件的一种，但是你不能确保在用户的点击过程中会发生一点点的滑动，这样就会造成子View的点击事件也可能会被拦截。因此Android提供了一个值，滑动距离小于这个值会被系统认为是点击，大于这个值系统会认为这是滑动。根据ROM的不同，这个值也会不同。就像上面代码中，我用`minDist`这个变量将值保存下来，获取这个值的方法是：`minDist = ViewConfiguration.get(context).getScaledTouchSlop()`。然后我们通过判断滑动的点的Y轴位置减去初始Y轴位置是否大于minDist来判断上拉还是下拉。最后说一下，Android好像提供了判断用户是上拉还是下拉的方法，我还没去研究，暂时先这样处理。还一个问题，我们怎么知道子View是否滑动到了顶部呢？我为此特意看了一下Android中SwipeRefreshLayout的源码，其中有一串代码如下：
```
private boolean canChildScrollUp() {
    return this.mTarget instanceof ListView ? ListViewCompat.canScrollList((ListView)this.mTarget, -1) : this.mTarget.canScrollVertically(-1);
}
```
这串代码就是判断子View是否能向上滚动，源码中还有一层我没摘录下来，我就觉得这段对我有用。
然后，我们还需要把子View保存下来，不然`this.mTarget`就是空指针，代码如下(SwipeRefreshLayout也是类似做法)：
```
private void ensureTarget() {
    if (this.mTarget == null) {
        final int count = this.getChildCount();
        for (int i = 0; i < count; i++) {
            View childView = getChildAt(i);
            if (!headerView.equals(childView)) {
                this.mTarget = childView;
                if (mTarget.getBackground() == null) {
                    mTarget.setBackgroundColor(Color.WHITE);
                }
            }
        }
    }
}

@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    this.ensureTarget();
}

@Override
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    super.onLayout(changed, left, top, right, bottom);
    this.ensureTarget();
}
```
截下来拦截下来的事件的处理了，从这一段开头理的逻辑中知道，下拉到一定位置才能触发刷新，如果处于刷新状态再下拉什么的就全部由子View处理：
```
public boolean onTouchEvent(MotionEvent event) {
    int action = event.getAction();
    if (action == MotionEvent.ACTION_MOVE) {
        dY = event.getY() - downY;
        if ((dY >= minDist && dY <= maxDY) && !isRefreshing) { // 如果没有正在刷新并且是下拉状态，并且没有超过最大下拉距离
            mTarget.setTranslationY(dY);
            if (dY > headerView.getMeasuredHeight()) { // 下拉距离超过headerView的高度，headerView在Y轴就要开始移动
                headerView.setTranslationY((dY - headerView.getMeasuredHeight()) / 2);
            }
            if (dY > refreshDist) { // 已经到了可以触发刷新的下拉距离
                configHeaderView(refreshOkText, CAN_REFRESH);
                canRefresh = true;
            } else { // 已经下拉但是还没到可以触发刷新的距离
                configHeaderView(readyText, READY_REFRESH);
                canRefresh = false;
            }
        }
    } else if (action == MotionEvent.ACTION_UP) { // 松手触发刷新
        if (dY > maxDY) {
            dY = maxDY;
        }
        if (dY > minDist) {
            if (!canRefresh && !isRefreshing) { // 如果还不能触发刷新并且没有正在刷新，松手的话就回弹回去
                ObjectAnimator.ofFloat(mTarget, "translationY", dY, 0).setDuration(500).start();
                ObjectAnimator.ofFloat(headerView, "translationY",
                        (dY - headerView.getMeasuredHeight()) / 2, 0)
                        .setDuration(500).start();
            } else if (!isRefreshing){ // 如果已经能触发刷新并且没有正在刷新，松手的话就回弹到最大距离的一半并且提示正在刷新
                ObjectAnimator.ofFloat(mTarget, "translationY", dY, refreshDist).setDuration(500).start();
                ObjectAnimator animator = ObjectAnimator.ofFloat(headerView, "translationY",
                        (dY - headerView.getMeasuredHeight()) / 2, (refreshDist - headerView.getMeasuredHeight()) / 2)
                        .setDuration(500);
                animator.addListener(new AnimatorListenerAdapter() {
                            @Override
                            public void onAnimationEnd(Animator animation) {
                                configHeaderView(refreshingText, ON_REFRESH);
                                isRefreshing = true;
                                canRefresh = false;
                            }
                        });
                animator.start();
            }
        }
        dY = 0;
    }
    return true;
}
```
最后，对外提供刷新完成的接口：
```
public void refreshFinish(boolean suc) {
    if (suc) {
        configHeaderView(refreshSuc, ON_FINISH, suc);
    } else {
        configHeaderView(refreshFail, ON_FINISH, suc);
    }
    // 刷新完成，提示刷新结果后
    ObjectAnimator animator1 = ObjectAnimator.ofFloat(mTarget, "translationY", refreshDist, 0);
    animator1.setDuration(200).setStartDelay(500);
    animator1.start();
    ObjectAnimator animator2 = ObjectAnimator.ofFloat(headerView, "translationY",
            (refreshDist - headerView.getMeasuredHeight()) / 2, 0);
    animator2.setDuration(200).setStartDelay(500);
    animator2.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            isRefreshing = false;
            canRefresh = false;
        }
    });
    animator2.start();
}
```
还有一些设置刷新监听的代码并没有放到本文中讲解，这一部分感觉很简单，可以到源码中查看更多详细内容，注释也都很详细。源码地址：[https://github.com/cyixlq/ViewTest](https://github.com/cyixlq/ViewTest)
##### 至此，一个简单的下拉刷新控件就完成了！这些代码肯定还是很繁琐，有很多有用的API我还没熟悉，希望以后能更进一步，把代码写的更精炼。