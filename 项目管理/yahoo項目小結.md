#Yahoo电商项目小结

## 概述
这是一个电商项目，我们主要负责移动端的开发，对接接口。这个项目让我在技术水平和项目流程管理规范上有了很大提升。

因为电商前后端都有许多逻辑，并且我们的后端出于架构考虑，把许多的逻辑都放在了前端来处理，这让前端打破了一直以来只是拿数据显示的简单逻辑。我们需要对数据分组、排序、重组等等做许多的操作，虽然这会占用大量资源。并且UI设计也是行业前沿的设计师，对许多UI上的要求也让我挺棘手的，但是最终都通过查阅大量资料实现了。下面我详细记录一下项目心得。

##1 项目管理
1. 减少开发中的问题-系统业务理解
2. 外包项目的客户交流方式
3. 协同开发的任务分配
4. 项目出现问题，此时开发人员不够，后期暂时又没有别的项目，招人？
5. 让别人干活干的爽
##2 实战技术
###2.1 窗口控件悬浮
###2.2 CardView在4.4以下的一些问题 shadownPadding

CardView 这个东西 慎用！UI的坑有点大。

在5.0 前后的机型上会有很大区别。5.0以上增加了Z坐标，阴影效果很好，5.0 之前的版本则是左上右下空出一部分区域用于绘制阴影。

这就造成有的UI可能是贴边的，但是这里被强制空出来一块，很难看。

API23以前可以使用setShadowPadding解决，但是之后的版本你回发现这个方法被内置了，CardView调用不到。就会抛出下面的错误
 
java.lang.NoSuchMethodError: android.support.v7.widget.CardView.setShadowPadding

所以我们想到可以改变他的MarginLayoutParmars，改为-3 就可以解决这个问题，但是注意在列表中，AbsListView的parmars没有继承MarginLayoutParmars

使用MarginLayoutParmars，但AbsListView 的LayoutParams没有继承这个

解决方案：https://android.jlelse.eu/using-full-width-cards-to-maximize-content-f739cb863ce8
```                                                        java.lang.IllegalArgumentException: pointerIndex out of range
```

重写onTouch 捕获这个异常

解决方案：https://github.com/chrisbanes/PhotoView/issues/31

###2.3 RecyleView上拉加载，下拉刷新
当时项目中RecycleView伴随5.0 才出来不久，还没有时间去研究，一直纠结于选择RecycleView还是使用传统ListView，处于对新事物的好奇，我选择了前者，但是也踩了超级多的坑。

####2.3.1 添加头尾

####2.3.2 布局切换

现在有很多成熟的库可以使用了，我目前用的是飘舟的[LRecycleView](https://github.com/jdsjlzx/LRecyclerView)

###2.4 Android Marital Design——theme
Android 5.0之后，UI上有了比较大的改变，主要的几点：
1. 沉浸式状态栏
2. 新增了Z轴的概念
###2.5 Android 位移动画
###2.6 AsyncTask 并行\串行任务
###2.7 软键盘弹出\关闭监听
###2.8 Activity进入退出动画效果
###2.9 android沉浸式状态栏
###2.10 ScrollView 的嵌套问题
###2.11 版本控制-Bitbucket
###2.12 7.0适配问题
1. 动态权限申请
2. FileProvider解决无法传递uri问题
3.  
 
##3 业务处理
1. 商品列表
2. 商品详情
3. 购买商品
4. 购物车
5. 结账
6. 购买记录

##4 编码体会
1. 一个方法只做单一的事
2. 逻辑相似的一起处理
3. 程序复用性
4. 程序健壮性