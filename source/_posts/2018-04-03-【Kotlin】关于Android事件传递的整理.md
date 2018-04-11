---
title: 【Kotlin】关于Android事件传递的整理
date: 2018-04-03 15:11:52
tags: 
- Kotlin
- 事件传递
---

关于事件传递的流程，已经有很多大神介绍过了，我在使用的过程中，也遇到了一些问题，在此整理一下，相信有不少同学也有遇到我这样的问题。

## 问题一：为什么我的`onTouchEvent`方法只响应了`MotionEvent.ACTION_DOWN`动作

百度或者Google一搜有一大把这样问题。其根本原因是你的`MotionEvent.ACTION_DOWN`行为没有被消费掉（没有`return true`）,`onTouchEvent`默认`MotionEvent.ACTION_DOWN`为一个触摸事件的开始，如果`MotionEvent.ACTION_DOWN`没有做处理，则后面一系列行为将都不被响应。

举一个例子，有一个嵌套布局

- A : ViewGroup
- B : ViewGroup
- C : View

> 注意，只有`ViewGroup`有`onInterceptTouchEvent`方法，View是没有`onInterceptTouchEvent`方法的，因为`View`不能再有子View，不涉及到事件传递

![ViewA->ViewB->ViewC](https://github.com/kongqw/kongqw.github.io/blob/next/source/img/Screenshot_1522379519.png?raw=true)

当我们手指触摸C控件的时候，事件的传递过程为

1. A - dispatchTouchEvent
2. A - onInterceptTouchEvent
3. B - dispatchTouchEvent
4. B - onInterceptTouchEvent
5. C - dispatchTouchEvent
6. C - onTouchEvent
7. B - onTouchEvent
8. A - onTouchEvent

> 以上为 dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent 均未重写的情况

默认情况下，你的`ACTION_DOWN`是没有被消费掉的，`super.onTouchEvent(event)`会返回`false`，比较常见的，有不少人会问，我手指触摸了C控件，但是为什么`ACTION_MOVE`和`ACTION_UP`没有响应呢，如果想要监听`ACTION_MOVE`和`ACTION_UP`，我们只需要重写`C`的`onTouchEvent`方法，在接收到`MotionEvent.ACTION_DOWN`的时候返回`true`，就代表这个触摸事件交给`C`处理了。

那么事件传递的过程将变成：

1. A - dispatchTouchEvent
2. A - onInterceptTouchEvent
3. B - dispatchTouchEvent
4. B - onInterceptTouchEvent
5. C - dispatchTouchEvent
6. C - onTouchEvent

就没有B、A 的`onTouchEvent`方法什么事了

## 问题二：手指在C控件上按下，我想要横着滑动的时候交给控件C处理，竖着滑动的时候交给B处理该如何实现

这里就涉及到事件的拦截处理，就轮到`onInterceptTouchEvent`方法大显身手的时候了。

首先聊一聊思路，我们都知道，事件的分发是由外到内的，即当我们在`C`控件上按下的时候，事件是先传到`A`上，`A`传给`B`，`B`再传给`C`（A -> B -> C）。事件处理是由内到外的，即`C`先处理，处理完再交给`B`，`B`处理完再交给`A`（C -> B -> A）。当前的需求涉及到B和C来处理滑动状态，那么，我们就可以在滑动事件分发到`B`的时候判断一下，当前的滑动是横向滑动的还是纵向滑动的：

1. 如果是横向滑动的，那么`onInterceptTouchEvent`方法不做拦截，允许事件传到`C`，`C`接收到事件后，在`onTouchEvent`中将事件消费掉，这样`B`就不需要进行处理（不会接收到onTouchEvent的响应）。
2. 如果是纵向滑动的，那么`onInterceptTouchEvent`就可以返回`true`，将事件拦截下来（不需要交给C处理，C就不会收的onTouchEvent的响应），`B`自己在`onTouchEvent`中将事件消费掉。


## 举个栗子

现在我们想实现这样一个功能，在页面上下滑动可以翻页，在控件上左右滑动可以拦截翻页动作

![这里写图片描述](https://github.com/kongqw/kongqw.github.io/blob/next/source/img/Screenshot_1522379272.gif?raw=true)

思路:

首先要写布局，自定义个 View 继承 `ViewGroup`, 然后开始我们的xml布局

``` XML
<com.kongqw.view.ScrollViewPager
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FFEEEEEE">

        <TextView
            android:id="@+id/tv1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:gravity="center"
            android:text="我是第 0 页内容"
            android:textSize="20sp" />

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_above="@id/tv1"
            android:layout_centerInParent="true"
            android:src="@mipmap/ic_launcher" />

    </RelativeLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FFBBBBBB">

        <Button
            android:id="@+id/btn_test"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:gravity="center"
            android:text="我是第 1 页内容"
            android:textSize="20sp" />

    </RelativeLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FF999999">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:text="我是第 2 页内容"
            android:textSize="20sp" />

        <SeekBar
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:background="#FF888888" />

    </RelativeLayout>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FF777777"
        android:gravity="center"
        android:text="我是第 3 页内容"
        android:textSize="20sp" />

</com.kongqw.view.ScrollViewPager>
```

写完布局我们要重写`onMeasure`和`onLayout`方法，开始对布局测量、排布。
我们这里是纵向排布。

``` Kotlin
/**
 * 测量
 */
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec)
    for (index in 0 until childCount) {
        measureChild(getChildAt(index), widthMeasureSpec, heightMeasureSpec)
    }
}

/**
 * 布局  纵向布局
 */
override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
    var top = 0
    var bottom = 0
    for (index in 0 until childCount) {
        val childView = getChildAt(index)
        bottom = top + childView.measuredHeight
        childView.layout(0, top, childView.measuredWidth, bottom)
        top = bottom
    }
    mBottomBorder = bottom
}
```

写到这里运行程序，应该是可以看得到第一个页面了，但是滑动没有任何反应，我们需要重写`onTouchEvent`方法，让View可以根据手指的滑动而滚动，这里需要监听event的`ACTION_DOWN`、`ACTION_MOVE`、`ACTION_UP`、`ACTION_CANCEL`状态，通过`ACTION_MOVE`记录手指每次纵向移动的距离，在监听到`ACTION_UP`、`ACTION_CANCEL`的时候，判断滑动位置，然后滚动到哪个页面。这样，一个简单的页面滑动就实现了。

### scrollBy、scrollTo

提到滚动，就不得不提到两个方法，`scrollBy`和`scrollTo`,值得注意的是，通常我们一种面向对象的思想，想让谁滚动，谁就调用自己的scroll就行了，但是，这里的`scrollBy`和`scrollTo`针对的是自己子View的运动，说白话了，就是老爸要管理儿子运动的。
`scrollBy`是控制一次滚动多少，`scrollTo`是控制滚动到哪里。

我们这里自定义了ScrollViewPager，页面内容都写在ScrollViewPager内部，所以，我们控制滚动，就是在控制ScrollViewPager内部的子View滚动，所以，直接调用`scrollBy`和`scrollTo`即可。

### Scroller

现在滑动没有问题了，但是手指一松开，View 会瞬间移动到指定位置，体验不太好，我们想要View的滚动有一个顺滑的过程，这就又不得不再提到一个`Scroller`类，他可以帮助我们完成动画的平缓过度。

通过`Scroller`的`startScroll`来初始化运动的起始位置、结束位置和运动时间，然后调用`invalidate()`开始处理，这里会一直回调`View`的`computeScroll`方法，是一个空方法，需要我们自己在这里自己依据`Scroller`的`computeScrollOffset()`判断移动是否完成，然后调用`scrollTo`来完成平滑移动处理。

到这里，View就可以实现上下滑动，并平滑过渡了，接下来来处理事件的分发问题。
我们想要实现的目标是，当我们上下滑动的时候，可以滚动页面，要将事件拦截。
当我们在子控件上左右滑动的时候，可以将事件先交给子控件处理。那么我们就可以重写`ScrollViewPager`的`onInterceptTouchEvent`方法，在`ACTION_MOVE`的时候判断滑动方向，如果是上下滑动，则返回true，将事件拦截即可。

下面是自定义`ScrollViewPager`源码（Kotlin代码）：

``` Kotlin
package com.kongqw.view.ScrollViewPager

import android.annotation.SuppressLint
import android.content.Context
import android.util.AttributeSet
import android.util.Log
import android.view.MotionEvent
import android.view.ViewConfiguration
import android.view.ViewGroup
import android.widget.Scroller

/**
 * Created by Kongqw on 2018/3/29.
 * ScrollViewPager
 */
class ScrollViewPager(context: Context?, attrs: AttributeSet?) : ViewGroup(context, attrs) {

    /**
     * 手指按下位置
     */
    private var mDownX = 0.0F
    private var mDownY = 0.0F

    /**
     * 手指移动后的位置
     */
    private var mMoveX = 0.0F
    private var mMoveY = 0.0F

    /**
     * 手指最后移动位置
     */
    private var mLastX = 0.0F
    private var mLastY = 0.0F

    /**
     * View 上边界
     */
    private val mTopBorder = 0

    /**
     * View 下边界
     */
    private var mBottomBorder = 0

    /**
     * 滑动处理
     */
    private val mScroller: Scroller = Scroller(context)

    /**
     * 事件是否拦截
     */
    private var isIntercept = false

    /**
     * 最小移动单位
     */
    private val mScaledtouchslop = ViewConfiguration.get(context).scaledTouchSlop

    companion object {
        private val TAG: String = ScrollViewPager::class.java.simpleName
    }

    /**
     * 测量
     */
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        for (index in 0 until childCount) {
            Log.i(TAG, "测量第 $index 个控件大小")
            measureChild(getChildAt(index), widthMeasureSpec, heightMeasureSpec)
        }
    }

    /**
     * 布局  纵向布局
     */
    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        var top = 0
        var bottom = 0
        for (index in 0 until childCount) {
            val childView = getChildAt(index)
            bottom = top + childView.measuredHeight
            Log.i(TAG, "布局第几个View [0, ${top}, ${childView.measuredWidth}, ${bottom}]")
            childView.layout(0, top, childView.measuredWidth, bottom)
            top = bottom
        }
        mBottomBorder = bottom
        Log.i(TAG, "View 高度 $height")
    }

    override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {

        val touchX = ev?.rawX
        val touchY = ev?.rawY

        when (ev?.action) {

            MotionEvent.ACTION_DOWN -> {
                mDownX = touchX!!
                mDownY = touchY!!
                isIntercept = false
            }
            MotionEvent.ACTION_MOVE -> {
                mMoveX = touchX!!
                mMoveY = touchY!!
                val dx = Math.abs(mMoveX - mDownX)
                val dy = Math.abs(mMoveY - mDownY)
//                if(dy > dx){
//                    isIntercept = true
//                }
                isIntercept = (dy > dx)
            }
            MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> {
                isIntercept = false
            }

        }

        mLastX = touchX!!
        mLastY = touchY!!

        return isIntercept
        // return super.onInterceptTouchEvent(ev)
    }

    @SuppressLint("ClickableViewAccessibility")
    override fun onTouchEvent(event: MotionEvent?): Boolean {

        val touchY = event?.rawY

        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {
                Log.i(TAG, "ACTION_DOWN")
                // 获取手指按下的位置
                mDownY = touchY!!
                return true
            }
            MotionEvent.ACTION_MOVE -> {
                Log.i(TAG, "ACTION_MOVE")
                // 获取手指移动后的位置
                mMoveY = touchY!!
                // 计算手指在Y轴移动距离
                // val dY = Math.abs(mLastY - mDownY)
                val dY = mLastY - mMoveY


                when {
                    scrollY < mTopBorder -> {
                        // 向下已经划出上边界
                        Log.i(TAG, "向下已经划出上边界")
                        scrollBy(0, dY.toInt() / 5)
                    }

                    scrollY + height > mBottomBorder -> {
                        // 向上已经划出下边界
                        Log.i(TAG, "向上已经划出下边界")
                        scrollBy(0, dY.toInt() / 5)
                    }

                    else -> {
                        // 移动
                        scrollBy(0, dY.toInt())
                    }
                }

                // 刷新页面
                invalidate()

                mLastY = mMoveY

            }
            MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> {
                Log.i(TAG, "ACTION_UP  View 纵向滑动了 $scrollY")

                when {
                    scrollY <= mTopBorder -> {
                        // 向下已经划出上边界
                        Log.i(TAG, "向下已经划出上边界")
                        mScroller.startScroll(0, scrollY, 0, -scrollY)
                        invalidate()
                    }

                    scrollY + height >= mBottomBorder -> {
                        // 向上已经划出下边界
                        Log.i(TAG, "向上已经划出下边界")
                        val dy = mBottomBorder - scrollY - height
                        mScroller.startScroll(0, scrollY, 0, dy)
                        invalidate()
                    }

                    else -> {
                        // 移动
                        val page = computePageByScrollY(scrollY)
                        Log.i(TAG, "滑动到第 $page 页")
                        mScroller.startScroll(0, scrollY, 0, height * page - scrollY)
                        invalidate()
                    }
                }
            }
        }

        mLastY = touchY!!

        return super.onTouchEvent(event)
    }

    override fun setOnTouchListener(l: OnTouchListener?) {
        super.setOnTouchListener(l)
    }

    /**
     * 重写平滑移动处理逻辑
     */
    override fun computeScroll() {
        super.computeScroll()
        if (mScroller.computeScrollOffset()) {
            // 滑动还没有结束
            scrollTo(mScroller.currX, mScroller.currY)
            invalidate()
        }
    }

    /**
     * 根据Y轴滑动距离判断应该显示哪一页
     */
    private fun computePageByScrollY(scrollY: Int = 0): Int {
        val v = scrollY / (height / 2)
        Log.i(TAG, "computePageByScrollY  scrollY = $scrollY   v = $v")
        return when (v) {
            0 -> 0
            1, 2 -> 1
            3, 4 -> 2
            5, 6, 7 -> 3
            else -> 0
        }
    }
}
```
