#概述
上一篇主要介绍了ViewAnim和帧动画，篇幅有点长，另起一篇。上篇介绍的两种动画开发中用到的不多，主要还是本篇的属性动画使用比较广。

## 1 补间动画

### 1.1 Property Anim

开发中Property Anim使用比View Anim要更为广泛，主要还是出于刚刚提到过的View Anim执行之后View的位置没有变化。
有的时候我们确实是需要改变View位置的。

#### 1.1.1 Object Anim

使用之前先介绍一些属性的含义

(1) translationX 和 translationY：这两个属性控制着 View 的屏幕位置坐标变化量，以 layout 容器的左上角为坐标原点;

(2) rotation、rotationX 和 rotationY：这三个属性控制着 2D 旋转角度（rotation属性）和围绕某枢轴点的 3D 旋转角度;

(3) scaleX、scaleY：这两个属性控制着 View 围绕某枢轴点的 2D 缩放比例;

(4) pivotX 和 pivotY: 这两个属性控制着枢轴点的位置，前述的旋转和缩放都是以此点为中心展开的,缺省的枢轴点是 View 对象的中心点;

(5) x 和 y：这是指 View 在容器内的最终位置，等于 View 左上角相对于容器的坐标加上 translationX 和 translationY 后的值;

(6)alpha：表示 View 的 alpha 透明度。缺省值为 1 （不透明），为 0 则表示完全透明（看不见）;

补间动画能实现的他都可以实现，举个栗子！

例1：从当前位置移动到Y轴300的位置
```
    private void startObjAnim() {
        // 第一个参数是执行动画的View，第二个参数是该View需要改变的属性，第三个参数是执行开始Y坐标，以屏幕坐标系为参照，第四个参数是移动到的位置
        ObjectAnimator objectAnimatorY = ObjectAnimator.ofFloat(mContentIv, "Y", mContentIv.getY(), 300);
        objectAnimatorY.setDuration(2000);//动画执行时间
        objectAnimatorY.setRepeatCount(2);//重复次数
        objectAnimatorY.setRepeatMode(ValueAnimator.REVERSE);//重复模式
        objectAnimatorY.setInterpolator(new BounceInterpolator());//插值器
        objectAnimatorY.start();
    }
```

这个例子不难看懂，但是如果想同时移动X,Y，并且要闪烁旋转呢？

例2：从当前位置移动到（100，300），移动中闪烁旋转

这里就不能再用AnimationSet了，因为ObjectAnimator是Animator的子类，我们使用AnimatorSet

```
    private void startObjAnim1() {
        AnimatorSet animatorSet = new AnimatorSet();
        ObjectAnimator objectAnimatorY = ObjectAnimator.ofFloat(mContentIv, "Y", mContentIv.getY(), 300);
        ObjectAnimator objectAnimatorX = ObjectAnimator.ofFloat(mContentIv, "X", mContentIv.getX(), 100);
        ObjectAnimator objectAnimatorRx = ObjectAnimator.ofFloat(mContentIv, "rotationX", 0f, 180f);
        ObjectAnimator objectAnimatorSx = ObjectAnimator.ofFloat(mContentIv, "scaleX", 1f, 0.5f);
        ObjectAnimator objectAnimatorA = ObjectAnimator.ofFloat(mContentIv, "alpha", 1f, 0.5f);
        animatorSet.play(objectAnimatorX).with(objectAnimatorY).with(objectAnimatorA).with(objectAnimatorSx).with(objectAnimatorRx);
        animatorSet.setDuration(3000);
        animatorSet.start();
    }
```

不难看懂，Animator一路构造，也可以指定在某个动画之前或之后调用，将with()换为befer()|after()。

之前在用ViewAnim做旋转的时候提了一个面试题，旋转之后保持现在的位置，那么用Property Anim就可以做了。

例3：播放按钮


```
	//开始
    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    private void startObjAnim2() {
        if (objectAnimatorR == null) {
            objectAnimatorR = ObjectAnimator.ofFloat(mContentIv, "rotation", 0f, 360f);
            objectAnimatorR.setDuration(3000);
            objectAnimatorR.setRepeatCount(3);
            objectAnimatorR.start();
        }else{
            objectAnimatorR.resume();
        }
    }

	//暂停
    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    private void switchAnimStop(String selectedItem) {
        if (objectAnimatorR != null && objectAnimatorR.isStarted()) {
            objectAnimatorR.pause();
        }
    }
```

主要就是Animator提供了一个pause和resume方法，简单看下内部做了什么事情。

```
    /**
     * 暂停运行中的动画，如果动画已经结束或还没开始，则该方法无效。
     * 可以通过调用resume()方法来继续执行动画
     */
    public void pause() {
        if (isStarted() && !mPaused) {
            mPaused = true;
            if (mPauseListeners != null) {
                ArrayList<AnimatorPauseListener> tmpListeners =
                        (ArrayList<AnimatorPauseListener>) mPauseListeners.clone();
                int numListeners = tmpListeners.size();
                for (int i = 0; i < numListeners; ++i) {
                    tmpListeners.get(i).onAnimationPause(this);
                }
            }
        }
    }
```

在执行pause的时候将mPaused置为true，动画暂停了，可以思考一下，动画执行中也是逐帧执行，应该是一个循环，并且还判断了当前的pause状态。那么我们接着看这个猜想是否正确，看下动画是如何执行的。

我们调用start方法，动画开始执行。

```
 private void start(boolean playBackwards) {
		...
		mStarted = true;
        mPaused = false;
        mRunning = false;
		...
        AnimationHandler animationHandler = AnimationHandler.getInstance();
        animationHandler.addAnimationFrameCallback(this, (long) (mStartDelay * sDurationScale));

        if (mStartDelay == 0 || mSeekFraction >= 0) {
            startAnimation();
            if (mSeekFraction == -1) {
                setCurrentPlayTime(0);
            } else {
                setCurrentFraction(mSeekFraction);
            }
        }
    }
```

这里去掉了一些代码和注释，我们想一下动画执行也是很多任务帧，应该也是一个线程去处理这个事情。

点进startAnimation和setCurrentFraction方法去看，都是一些设置isRunning，mSeekFraction等一些值的操作，没有看到Thread，Runnable这些的影子。于是我们看下AnimationHandler.addAnimationFrameCallback

```
    public void addAnimationFrameCallback(final AnimationFrameCallback callback, long delay) {
        if (mAnimationCallbacks.size() == 0) {
            getProvider().postFrameCallback(mFrameCallback);
        }
        if (!mAnimationCallbacks.contains(callback)) {
            mAnimationCallbacks.add(callback);
        }

        if (delay > 0) {
            mDelayedCallbackStartTime.put(callback, (SystemClock.uptimeMillis() + delay));
        }
    }
```

mAnimationCallbacks这个明显是个集合放了很多监听器，我们在第一次调用的时候走的应该是mAnimationCallbacks.size() == 0这个逻辑，那么下面那个Provider是个什么鬼？？

点进去看他new 了一个MyFrameCallbackProvider，这里面有一个类Choreographer，在点下去看！

介绍是 * Coordinates the timing of animations, input and drawing.关联了动画时间，输出和绘制！好像是我们在找的东西！再往下看他的介绍，确认了他就是让动画执行的类。

啊哈！看到了ThreadLocal，终于被我们找到了！但是并不是我们需要的。在看，postFrameCallback，一路点下去有个doCallbacks的方法，看一下。

```
 void doCallbacks(int callbackType, long frameTimeNanos) {
        CallbackRecord callbacks;
        	... //这段去掉，我们只看CALLBACK_ANIMATION的部分

        try {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, CALLBACK_TRACE_TITLES[callbackType]);
            for (CallbackRecord c = callbacks; c != null; c = c.next) {
                c.run(frameTimeNanos);
            }

        } ...//省略了finally处理，
    }
```

看到调用了一个run方法，点进去看下，看到他调用了传入回调的doFrame方法回传了frameTimeNanos，如果不是FRAME_CALLBACK_TOKEN则调用runnable的run

计算完grameTimeNanos后回调给了AnimatorHandler执行doAnimationFrame,并继续下一帧的计算

```
private final Choreographer.FrameCallback mFrameCallback = new Choreographer.FrameCallback() {
        @Override
        public void doFrame(long frameTimeNanos) {
            doAnimationFrame(getProvider().getFrameTime());
            if (mAnimationCallbacks.size() > 0) {
                getProvider().postFrameCallback(this);
            }
        }
    };
```
看下doAnimationFrame回调到了ValueAnimtor里面的方法

```
    public final void doAnimationFrame(long frameTime) {
       ...
        if (mPaused) {
            mPauseTime = frameTime;
            handler.removeCallback(this);
            return;
        } else if (mResumed) {
            mResumed = false;
            if (mPauseTime > 0) {
                // Offset by the duration that the animation was paused
                mStartTime += (frameTime - mPauseTime);
                mStartTimeCommitted = false; // allow start time to be compensated for jank
            }
            handler.addOneShotCommitCallback(this);
        }
      ...
    }
```

终于被我们找到了如果是暂停状态则移除回调监听，不会在绘制，当我们在调用resume的时候有添加了callback继续绘制，直到动画完成。

不知不觉看了有点多，小结一下

- ObjectAnim是通过线程计算每一帧绘制的
- 在调用pause时候任务还在进行，但是不会走回调显示的方法
- 重新resume之后接着上次的位置继续绘制直到动画结束

因此我们的需求可以满足，也可以看出来其实pause之后动画任务还是在消耗资源

#### 1.1.2 ValueAnimtor实现动画

经过上面的分析也不难看出ValueAnimtor是ObjAnim的父类，他们都是Animtor的子类。当然使用ObjectAnimtor要比ValueAnimtor方便，这里也介绍下ValueAnimtor是怎么用的吧~

例1:将View缩放到自身的一半，用ValueAnimtor实现

```
    private void startValueAnim1() {
        //1. 创建句柄
        PropertyValuesHolder scaleX = PropertyValuesHolder.ofFloat("scaleX", 1f, 0.5f);
        PropertyValuesHolder scaleY = PropertyValuesHolder.ofFloat("scaleY", 1f, 0.5f);
        ValueAnimator valueAnimator = ValueAnimator.ofPropertyValuesHolder(scaleX, scaleY);
        //2. 设置属性变化监听
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //3. 获取变化值并设置给View
                float animatorValueScaleX =  (float) animation.getAnimatedValue("scaleX");
                float animatorValueScaleY = (float) animation.getAnimatedValue("scaleY");
                mContentIv.setScaleX(animatorValueScaleX);
                mContentIv.setScaleY(animatorValueScaleY);
            }
        });
        valueAnimator.setDuration(2000);
        valueAnimator.setRepeatCount(2);
        valueAnimator.setRepeatMode(ValueAnimator.REVERSE);
        valueAnimator.start();
    }
```

### 1.2 Layout Anim

除了上述的几种动画外，还有一种动画，在开发中也遇到了，下面也介绍一下布局动画。

#### 1.2.1 LayoutAnimation

LayoutAnimation 是API Level 1 就已经有的，LayoutAnimation是对于ViewGroup控件所有的child view的操作，也就是说它是用来控制ViewGroup中所有的child view 显示的动画。

<b>XML实现方式</b>

1.在res/anim包下面申明动画效果 anim_top_to_down.xml

```
<?xml version="1.0" encoding="utf-8"?>
<!--向下移动到界面一半并透明-->
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:repeatMode="reverse"
     android:duration="3000">
    <translate
        android:fromYDelta="0"
        android:toYDelta="50%"/>
    <alpha
        android:fromAlpha="1"
        android:toAlpha="0.5"/>
</set>
```

2.在res/anim包下声明anim_cust_layout.xml

```
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
                 android:animation="@anim/anim_top_to_down"
                 android:animationOrder="reverse"
                 android:delay="30%"/>
```

3.在布局文件中添加

```    android:layoutAnimation="@anim/anim_cust_layout"
```

<b>代码实现</b>

```
   //通过加载XML动画设置文件来创建一个Animation对象；
   Animation animation=AnimationUtils.loadAnimation(this, R.anim.slide_right);   //得到一个LayoutAnimationController对象；
   LayoutAnimationController controller = new LayoutAnimationController(animation);   //设置控件显示的顺序；
   controller.setOrder(LayoutAnimationController.ORDER_REVERSE);   //设置控件显示间隔时间；
   controller.setDelay(0.3);   //为ListView设置LayoutAnimationController属性；
   listView.setLayoutAnimation(controller);
   listView.startLayoutAnimation();
```

#### 1.2.2 LayoutTransition

LayoutTransition 是API Level 11 才出现的。LayoutTransition的动画效果，只有当ViewGroup中有View添加、删除、隐藏、显示的时候才会体现出来。

- 在xml中```android:animateLayoutChanges="true"```
- 为ViewGroup添加动画 ```LayoutTransition mTransitioner = new LayoutTransition();  
mViewGroup.setLayoutTransition(mTransitioner);  ```   

其实也是ObjectAnimtor实现的。

学习的时候在网上看到一个例子，也记录一下。

原文地址：[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0619/3090.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0619/3090.html)

文中是使用布局动画改编每个布局的播放顺序实现的。我们换一种方式

使用了ObjAnim的方式，自己感觉这样确实很蠢，使用了动画延迟

```
    public void scaleView(List<View> vs) {
        int l = 0;
        AnimatorSet set = new AnimatorSet();
        for (int i = 0; i < vs.size(); i++) {
            View v = vs.get(i);
            if (i == vs.size() / 2)
                l = 100;
            l += 100;
            AnimatorSet itemSet = new AnimatorSet();
            ObjectAnimator scaleX = ObjectAnimator.ofFloat(v, "scaleX", 1f, 0f);
            scaleX.setRepeatMode(ValueAnimator.RESTART);
            ObjectAnimator scaleY = ObjectAnimator.ofFloat(v, "scaleY", 1f, 0f);
            scaleY.setRepeatMode(ValueAnimator.RESTART);
            ObjectAnimator alpha = ObjectAnimator.ofFloat(v, "alpha", 1f, 0f);
            alpha.setRepeatMode(ValueAnimator.RESTART);
            itemSet.play(scaleX).with(scaleY).with(alpha);
            itemSet.setDuration(250);
            itemSet.setStartDelay(l);
            set.play(itemSet);
        }
        set.start();
    }
```

### 1.3 插值器（Interpolator）

计算运行速度的类。

#### 1.3.1 常见插值器

- AccelerateDecelerateInterpolator 在动画开始与介绍的地方速率改变比较慢，在中间的时候加速
- AccelerateInterpolator 在动画开始的地方速率改变比较慢，然后开始加速
- AnticipateInterpolator 开始的时候向后甩一点然后向前
- AnticipateOvershootInterpolator 开始的时候向后甩一点然后向前超过设定值一点然后返回
- BounceInterpolator 动画结束的时候弹起，类似皮球落地
- CycleInterpolator 动画循环播放特定的次数回到原点，速率改变沿着正弦曲线
- DecelerateInterpolator 在动画开始的地方快然后慢
- LinearInterpolator 以常量速率改变
- OvershootInterpolator 向前超过设定值一点然后返回 

此处参考：[http://blog.csdn.net/daydayplayphone/article/details/52503665](http://blog.csdn.net/daydayplayphone/article/details/52503665)

有图可以自己去看更直观，其实很简单，常用的记住就好，英文意思很明确了

下面我们看下如何自己定义一个插值器！

#### 1.3.2自定义插值器

自定义插值器需要实现 Interpolator / TimeInterpolator接口 & 复写getInterpolation（）
 
- 补间动画 实现 Interpolator接口；属性动画实现TimeInterpolator接口 
- TimeInterpolator接口是属性动画中新增的，用于兼容Interpolator接口，这使得所有过去的Interpolator实现类都可以直接在属性动画使用

```
  /**
     * 插值分数
     * @param input 值在0~1之间，表明当前点在动画中的位置，0是起始点，1是结束点
     * @return 插值分数< 0低于目标，>1则超过目标
     */
    float getInterpolation(float input);
```

### 1.4 估值器（TypeEvaluator）

作用：设置 属性值 从初始值过渡到结束值 的变化具体数值 

- 插值器（Interpolator）决定 值 的变化规律（匀速、加速blabla），即决定的是变化趋势；而接下来的具体变化数值则交给估值器 
- 协助插值器 实现非线性运动的动画效果

```
// 在第4个参数中传入对应估值器类的对象
ObjectAnimator anim = ObjectAnimator.ofObject(myView2, "height", new Evaluator()，1，3);
```

系统内置的估值器有3个：
- IntEvaluator：以整型的形式从初始值 - 结束值 进行过渡
- FloatEvaluator：以浮点型的形式从初始值 - 结束值 进行过渡
- ArgbEvaluator：以Argb类型的形式从初始值 - 结束值 进行过渡

*注 那么插值器的input值 和 估值器fraction有什么关系呢？

答：input的值决定了fraction的值：input值经过计算后传入到插值器的getInterpolation（），然后通过实现getInterpolation（）中的逻辑算法，根据input值来计算出一个返回值，而这个返回值就是fraction了