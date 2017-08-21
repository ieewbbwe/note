#概述
Android开发中一直会遇到各种动画效果，特别是如果老板和UI妹子很扣这块的话。这也是每一个Android程序员无法绕过的一块内容，目前项目不忙，终于有时间来系统的整理下Android中的动画。

Android中动画分为:

1. 帧动画（Frame Anim），像幻灯片那样逐张播放
2. 补间动画(Tween Anim)，比如对一个TextView执行一系列简单变换（位置、大小、旋转、透明度）

还有属性动画（Property Anim），布局动画（Layout Anim），都归在补间动画里下面会介绍
## 1 帧动画
帧动画有两种实现方式1.xml中引用drawable 2.代码实现，下面就这两种实现方式我们分别写Demo看下

### 1.1 xml中实现

帧动画按照在animation-list中的item逐个播放，每次开始播放都会从第一帧开始。

### 1.2 代码实现
思路也是给控件设置Drawable,逐帧播放。效果和上面一样的就不贴图了。

```
 AnimationDrawable animDrawableBg = new AnimationDrawable();
        animDrawableBg.addFrame(getResources().getDrawable(R.mipmap.wifi1), 500);
        animDrawableBg.addFrame(getResources().getDrawable(R.mipmap.wifi2), 500);
        animDrawableBg.addFrame(getResources().getDrawable(R.mipmap.wifi3), 500);
        animDrawableBg.addFrame(getResources().getDrawable(R.mipmap.wifi4), 500);
        animDrawableBg.addFrame(getResources().getDrawable(R.mipmap.wifi5), 500);
        animDrawableBg.addFrame(getResources().getDrawable(R.mipmap.wifi6), 500);
        animDrawableBg.setOneShot(false);//设置循环播
        mContentIv.setImageDrawable(animDrawableBg);
```

## 2 补间动画
在说动画之前，我们要先了解下Android中的坐标系。

首先来看以屏幕坐上角为原点的坐标系，是Android坐标系也可以叫屏幕坐标系。

其次看以视图View左上角为原点的坐标系叫视图坐标系

### 2.1 View Anim
#### 2.1.1 位移动画（Translate）
把布局从一个坐标位移动到另一个坐标位。

```
TranslateAnimation(Context context, AttributeSet attrs)

TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta) 

TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)

```

有三个构造方法，我们直接看参数最多的那个。

```
 /**
     * 位移动画构造方法
     *
     * @param fromXType  X轴方向的参照，有如下几个参数可选，默认是ABSOLUTE
     *                   Animation.ABSOLUTE,相对于具体坐标系值，比如100到300，指绝对的屏幕像素单位
     *                   Animation.RELATIVE_TO_SELF，相对于自身，移动倍数
     *                   Animation.RELATIVE_TO_PARENT吗，相对于父容器，移动倍数
     * @param fromXValue 动画开始的X轴值，以视图坐标系为参照，不是以屏幕的坐标系
     * @param toXType X轴终点参照
     * @param toXValue 动画结束的X轴值
     * @param fromYType Y轴方向的参照系，参数同X轴一样
     * @param fromYValue 动画开始的Y轴值
     * @param toYType Y轴终点参照
     * @param toYValue 动画结束的Y轴值
     */
    public TranslateAnimation(int fromXType, float fromXValue, int toXType, float toXValue,
                              int fromYType, float fromYValue, int toYType, float toYValue)
```

举几个栗子！

例子1：把View从（0,0）移动到（200,200）

```
private void translate() {
        TranslateAnimation translateAnim = new TranslateAnimation(0, 200, 0, 200);
        translateAnim.setDuration(2000);//动画执行时间
        mAnimIv.startAnimation(translateAnim);//开始动画
    }
```

例子2：把View从（0,0）移动到（2倍自身的X，2倍自身的Y）

```
 private void translate() {
       TranslateAnimation translateAnim = new TranslateAnimation(Animation.RELATIVE_TO_SELF, 0, Animation.RELATIVE_TO_SELF, 2,
                Animation.RELATIVE_TO_SELF, 0, Animation.RELATIVE_TO_SELF, 2);
        translateAnim.setDuration(2000);//动画执行时间
        translateAnim.setRepeatCount(1);//重复两次
        translateAnim.setRepeatMode(Animation.REVERSE);//反向
        mAnimIv.startAnimation(translateAnim);//开始动画
    }
```

介绍一下RepeatMode，动画的重复模式

```
  /**
     * 定义动画的重复模式，只在RepeatCount>0 的情况下有效
     *
     * @param repeatMode 重复模式
     *                   RESTART，默认值，动画结束回到初始位置
     *                   REVERSE，动画结束从结束点在倒退回原始位置
     */
    public void setRepeatMode(int repeatMode)
```

#### 2.1.2 缩放动画（Scale）

```
    public ScaleAnimation(Context context, AttributeSet attrs) 

	public ScaleAnimation(float fromX, float toX, float fromY, float toY)

	public ScaleAnimation(float fromX, float toX, float fromY, float toY,
            float pivotX, float pivotY)

	public ScaleAnimation(float fromX, float toX, float fromY, float toY,
            int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
```

看下他第三个构造方法，其余类似

```
 /**
     * 缩放动画构造器
     *
     * @param fromX 起始位横向缩放因子
     * @param toX 结束位横向缩放因子
     * @param fromY 起始位纵向缩放因子
     * @param toY 结束位纵向缩放因子
     * @param pivotX 缩放中心点X轴坐标
     * @param pivotY 缩放中心点Y轴坐标
     */
    public ScaleAnimation(float fromX, float toX, float fromY, float toY,
                          float pivotX, float pivotY)
```

例子1：将View以左上角为缩放点放大一倍

```
 private void scale() {
        scaleAnim = new ScaleAnimation(1, 2, 1, 2, 0, 0);
        scaleAnim.setDuration(2000);
        scaleAnim.setRepeatCount(1);
        scaleAnim.setRepeatMode(Animation.REVERSE);
        mAnimIv.startAnimation(scaleAnim);
    }
```

例子2：将View以中心为缩放点 缩小一倍

```
 private void scale() {
        scaleAnim = new ScaleAnimation(1, 0.5f, 1, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
        scaleAnim.setDuration(2000);
        scaleAnim.setRepeatCount(1);
        scaleAnim.setRepeatMode(Animation.REVERSE);
        mAnimIv.startAnimation(scaleAnim);
    }
```

#### 2.1.3 旋转动画（Rotate）

```
    public RotateAnimation(Context context, AttributeSet attrs) 
 	
	public RotateAnimation(float fromDegrees, float toDegrees)

    public RotateAnimation(float fromDegrees, float toDegrees, float pivotX, float pivotY) 

 	public RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue,
            int pivotYType, float pivotYValue) 
```

一样，我们只看参数最多的那个构造。

```
/**
     * 旋转动画的构造
     *
     * @param fromDegrees 动画开始时的角度
     * @param toDegrees   动画结束时的角度
     * @param pivotXType  X轴起始点类型，有如下几种可选，默认是 Animation.ABSOLUTE
     *                    Animation.ABSOLUTE，像素值
     *                    Animation.RELATIVE_TO_SELF，以自身为参照，倍数
     *                    Animation.RELATIVE_TO_PARENT，以父容器为参照，倍数
     * @param pivotXValue X轴起始点值
     * @param pivotYType  Y轴起始点类型
     * @param pivotYValue Y轴起始点值
     */
    public RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue,
                           int pivotYType, float pivotYValue)

```
举个栗子！

例子1：以初始点为旋转点，将View旋转360度

(这里为了旋转时转出屏幕外，我先将View设置了layout_grivity:'center')

```
private void rotate() {
        rotateAnim = new RotateAnimation(0, 360);
        rotateAnim.setDuration(2000);
        rotateAnim.setRepeatCount(1);
        rotateAnim.setRepeatMode(Animation.REVERSE);
        mAnimIv.startAnimation(rotateAnim);
    }
```

例子2：以控件中心点位旋转点，将View旋转360度

```
private void rotate() {
        rotateAnim = new RotateAnimation(0, 360, Animation.RELATIVE_TO_SELF, 1.5f, Animation.RELATIVE_TO_SELF, 1.5f);
        rotateAnim.setDuration(2000);
        rotateAnim.setRepeatCount(1);
        rotateAnim.setRepeatMode(Animation.REVERSE);
        mAnimIv.startAnimation(rotateAnim);
    }
```

当然也可以以随便一个点为旋转点画圆，自己去试吧~

<b>*补充</b> 在这里突然想到以前一个面试题。

问题：在做旋转的时候暂停，将View固定为暂停时的角度，下次开始时就从上次暂停的角度开始。

说的具体一点，就类似以前的碟片机那样，自己脑补吧。

这个需求用View Anim还不行，因为动画取消后，View就回归到了原始的位置。如图：

这也就迁出了View Anim的另一个特点，他改变的并不是View在屏幕上真正的位置。

对应这个特点，有一个bug，在View执行时的点击问题。

我首先给View添加一个点击事件,就是弹一个Toast，下面看问题

这里我点击移动中的View是不会触发点击事件的，但是我点击View最开始的位置就触发了点击事件！

#### 2.1.4 透明动画（Alpha）

```
    public AlphaAnimation(Context context, AttributeSet attrs)

    public AlphaAnimation(float fromAlpha, float toAlpha)

```

这个动画虽然简单，但是使用也是很多的，比如做一个闪光效果，透明效果都会用到

```
    /**
     * 透明动画构造器
     * @param fromAlpha 动画开始的透明值0~1 0表示完全透明，1表示不透明
     * @param toAlpha 动画结束的透明值
     */
    public AlphaAnimation(float fromAlpha, float toAlpha)
```

例子1：制作一个闪动的View

```
 private void alpha() {
        alphaAnim = new AlphaAnimation(1, 0f);
        alphaAnim.setDuration(800);
        alphaAnim.setRepeatCount(4);
        alphaAnim.setRepeatMode(Animation.REVERSE);
        mAnimIv.startAnimation(alphaAnim);
    }
```

#### 2.1.5 组合动画（）

除了上面四种外，还可以把他们互相组合起来用

```
    /**
     * 组合动画构造器
     *
     * @param shareInterpolator True:set里面所有动画使用set统一的插值器 False:使用自己各自的插值器
     */
    public AnimationSet(boolean shareInterpolator)
```

例子1：渐变向下移动

```
    private void animSet() {
        AnimationSet animationSet = new AnimationSet(true);
        TranslateAnimation translateAnimation = new TranslateAnimation(Animation.RELATIVE_TO_PARENT, 0, Animation.RELATIVE_TO_PARENT, 0f,
                Animation.RELATIVE_TO_PARENT, 0f, Animation.RELATIVE_TO_PARENT, 0.8f);
        translateAnimation.setDuration(3000);
        AlphaAnimation alphaAnimation = new AlphaAnimation(1, 0.5f);
        alphaAnimation.setDuration(3000);
        animationSet.addAnimation(translateAnimation);
        animationSet.addAnimation(alphaAnimation);
        animationSet.setInterpolator(new BounceInterpolator());//插值器
        mAnimIv.startAnimation(animationSet);
    }
```

弹起的效果是插值器做的，我们介绍完动画接下来就介绍他！

<b>*注意</b> 动画时间会以animationSet设定的时间为准，如果没设置则会以内部动画设置的时间为准

### 2.2 Property Anim

开发中Property Anim使用比View Anim要更为广泛，主要还是出于刚刚提到过的View Anim执行之后View的位置没有变化。
有的时候我们确实是需要改变View位置的。

#### 2.2.1 Object Anim

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

- ObjectAnim是通过XX绘制的
- 在调用pause时候任务还在进行，但是不会走回调显示的方法
- 重新resume之后接着上次的位置继续绘制直到动画结束

因此我们的需求可以满足，也可以看出来其实pause之后动画任务还是在消耗资源

#### 2.2.2 ValueAnimtor实现动画

经过上面的分析也不难看出


### 2.3 Layout Anim
### 2.4 插值器（interpolator）
