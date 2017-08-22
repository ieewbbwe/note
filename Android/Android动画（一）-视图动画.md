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

