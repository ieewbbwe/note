##1 前言
最近项目上有一个跑马灯的需求。</br>

需求：

- 无限滚动，可以自动切换下一条
- 如果当前的文本超过一屏，则滚动完当前再切换下一条

第一点很简单，但是第二点就比较蛋疼了，看了网上很多轮子都没有太合适的，于是自己写了一个。 记录总结一下Android 跑马灯的实现方式，和我自定义跑马灯的思路。
![](https://i.imgur.com/XmLwBXN.gif)

##2 内容
###2.1 需求实现
#### 2.1.1 使用RecycleView + 自定义跑马灯
	思路:
	1. 利用橫向RecycleView实现排列
	2. smoothScroll实现滑动 
	3. 内嵌跑马灯实现标题超长的滚动

#### 2.1.4 ViewFlipper + 自定义跑马灯
这个实现方式是好基友告诉我的，很nice了，原理也是嵌套。
因为使用了ViwFilpper 因此想要改变滑动方向很简单。直用该进入\退出动画即可</br>

	思路：
	1. 外层使用ViewFlipper切分View并且滚动
	2. 内层使用自定义跑马灯滚动

#### 2.1.3 自定义控件，Draw() 出滚动效果
	思路
	1. draw出前后两笔文本
	2. 利用view的重绘，不断更新文本位置实现滚动


###2.2 跑马灯的实现方式
####2.2.1 TextView 设置Marquee
```
<TextView
        android:id="@+id/marqueeNormal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:ellipsize="marquee"
        android:focusable="true"
        android:focusableInTouchMode="true"
        android:marqueeRepeatLimit="marquee_forever"
        android:singleLine="true"
        android:text="我是普通的TextView跑马灯~跑啊跑~！" />
```

灰常简单，主要是这几个熟悉：
```
 android:ellipsize="marquee"
 android:marqueeRepeatLimit="marquee_forever"
 android:singleLine="true"
```

但是！！有些时候会无效！因为跑马灯要跑起来需要获取到焦点，但是由于界面复杂，有时候焦点你好控制！TextView声明只有再isFocus的时候才会走跑马灯，具体的源码自己去看了。

那么这种情况的终极解决方案是！直接继承TextView，并Override isFocused()，永久返回True。就可以了，但是还会有弹出Dialog，屏幕不亮等时候会出现问题。

<b>最终解决如下，实测可用：</b>

```
public class MarqueeText3 extends AppCompatTextView {
    public MarqueeText3(Context context) {
        this(context,null);
    }

    public MarqueeText3(Context context, AttributeSet attrs) {
        super(context, attrs);
        //设置单行
        setSingleLine();
        //设置Ellipsize
        setEllipsize(TextUtils.TruncateAt.MARQUEE);
        //获取焦点
        setFocusable(true);
        //走马灯的重复次数，-1代表无限重复
        setMarqueeRepeatLimit(1);
        //强制获得焦点
        setFocusableInTouchMode(true);

    }

    @Override
    public boolean isFocused() {
        return true;
    }

    @Override
    protected void onFocusChanged(boolean focused, int direction, Rect previouslyFocusedRect) {
        if (focused) {
            super.onFocusChanged(focused, direction, previouslyFocusedRect);
        }
    }

    @Override
    public void onWindowFocusChanged(boolean hasWindowFocus) {
        if (hasWindowFocus)
            super.onWindowFocusChanged(hasWindowFocus);
    }

}
```

参考：[TextView中的跑马灯不动](https://blog.csdn.net/zinjin_woxin/article/details/50370897)

####2.2.2 自定义控件-使用ViewFlipper
####2.2.3 自定义控件-使用Scroller 滚动
####2.2.4 自定义控件-使用ScrollTo()
####2.2.5 自定义控件-Canvas Draw()

##3 总结
<b>总的来说，就两种思路：</b> 

1. 外层滚动嵌套内层滚动
2. 绘制当前和下一个，滚动

<b>使用手感：</b>

使用自定义Draw的方式 可控性高，灵活，但是容易出错，计算方式比较麻烦，例子中写的还不完善；

使用嵌套的方式简单，快速，可控性不高，比如滚动时的停顿时间。
 
<b>点击事件</b>

还有一个需要注意的是，draw方法的时候，因为没有控制好点击的问题，必须再切换的时候做到迅速，不然会有一个错误情况：当第一条未滚动出屏幕，但第二条一家滑入一部分的时候，点击会出现错误，不清楚当前是属于第一条还是第二条。

参考：

[https://github.com/sunfusheng/MarqueeView](https://github.com/sunfusheng/MarqueeView)

[https://github.com/shenjiajun53/CustomizedViews](https://github.com/shenjiajun53/CustomizedViews)