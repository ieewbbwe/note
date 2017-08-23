# 概述

上两篇介绍了一些动画基础

 [ Android动画（一）-视图动画](http://blog.csdn.net/moxiouhao/article/details/77481723)

 [Android动画（二）-属性动画](http://blog.csdn.net/moxiouhao/article/details/77482082)

但是开发中为了开发效率，我们通常是使用一些三方的库，有前辈已经封装了很完善的动画库，我们学习一下直接用，使用中还能探寻框架作者的设计思路，事半功倍，何乐不为~

## 1 常见动画框架

### 1.1 AndroidViewAnimations

NineOldAnimations一个老牌动画开源库了，JakeWharton大神的作品，大神提供的是框架类的思路，没有Demo，自己用起来还是蛮吃力的，那么这个事情已经有好人帮我们做了！

代码家的AndroidViewAnimations，Github地址：[https://github.com/daimajia/AndroidViewAnimations
](https://github.com/daimajia/AndroidViewAnimations)

他封装了NineOld，并提供了一大包的动画效果可以供使用者选用。简单看下他的Demo。

```
  mListView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
				//1 停掉上一个动画
                if (rope != null) {
                    rope.stop(true);
                }

                Techniques technique = (Techniques) view.getTag();
				//2 动画工厂生产YoYo
                rope = YoYo.with(technique)
                        .duration(1200)
                        .playOn(mTarget);
            }
        });
```

Techniques是一个枚举，列出了目前支持的动画。工厂类根据这个枚举创建出不同动画的类。playOn()创建了一个控制类把动画包装起来进行控制，并开启这个动画。

主要的还是枚举里面列出的动画条目，随便看一个类吧

```
public class ZoomOutRightAnimator extends BaseViewAnimator {
    @Override
    protected void prepare(View target) {
        ViewGroup parent = (ViewGroup) target.getParent();
        int distance = parent.getWidth() - parent.getLeft();
        getAnimatorAgent().playTogether(
                ObjectAnimator.ofFloat(target, "alpha", 1, 1, 0),
                ObjectAnimator.ofFloat(target, "scaleX", 1, 0.475f, 0.1f),
                ObjectAnimator.ofFloat(target, "scaleY", 1, 0.475f, 0.1f),
                ObjectAnimator.ofFloat(target, "translationX", 0, -42, distance)
        );
    }
}
```

它内部也是使用AnmitorSet包裹了属性动画，并提供了一个动画基类统一控制。

## 2 原理解析
占个坑！