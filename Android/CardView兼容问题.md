
## 概述

开发中以前用的编译版本是23，最近升级到了25.3.0，于是就出现了这个蛋疼的问题。java.lang.NoSuchMethodError: android.support.v7.widget.CardView.setShadowPadding

###问题分析：

出现这个问题的人必然是发现使用过了CardView 在LOLLIPOP版本以下出现的兼容性问题，导致了阴影部分出现了很大的间隙。想要手动的取消或者减小这个间隙。如下图：

CardView 是LOLLIPOP之后出现的新控件，主要效果就是阴影，原理是21之后图层引入了Z轴的概念，但是21之前的版本没有Z轴的概念，于是Google就通过shadowBound 空出一块区域来绘制阴影。

19版本下的阴影

23版本下的阴影

看一下源码发现

25的编译版本

```
private final CardViewDelegate mCardViewDelegate = new CardViewDelegate() {
  ...
   @Override
   public void setShadowPadding(int left, int top, int right, int bottom) {
        mShadowBounds.set(left, top, right, bottom);
        CardView.super.setPadding(left + mContentPadding.left, top + mContentPadding.top,
                    right + mContentPadding.right, bottom + mContentPadding.bottom);
        }
  ...
}
```

21的编译版本

```
**
     * Internal method used by CardView implementations to update the padding.
     *
     * @hide
     */
    @Override
    public void setShadowPadding(int left, int top, int right, int bottom) {
        mShadowBounds.set(left, top, right, bottom);
        super.setPadding(left + mContentPadding.left, top + mContentPadding.top,
                right + mContentPadding.right, bottom + mContentPadding.bottom);
    }
```

看到了，高版本的setShadowPadding被隐藏到一个内部类里面了，用CardView是不能直接调用的，其实一早也就给了hide标记

###解决办法:设置负的margin抵消ShadowPadding

1 在value的dimens 里面定义边距为-4dp 

2 新建value-21在dimens里面定义边距为0

	android:layout_marginLeft="-4dp"

尝试过反射设置mShadowBounds；重新兴建一个阴影图层等方法之后发现，设置margin最简单，新建阴影图层方法效果好，但是有间隔

大神们推荐的解决方案是设置margin：https://android.jlelse.eu/using-full-width-cards-to-maximize-content-f739cb863ce8