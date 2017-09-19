#概述
之前公司招Android开发的一些问题

##面试问题

###1. Android四大组件是什么？及各自的生命周期？
四大组件：Activity，Service，BrodcastReceiver，ContentProvider

<b>Activity</b>

介绍：通常在项目中负责展示页面，我们看到的每一个页面一般都是一个Activity,也可能是fragment。

生命周期：onCreate->onStart->onResume->onPause-onStop->onDestroy

衍生问题：
####Activity是如何被创建的？
调用startActivity()->
####在界面被退到后台在唤醒会有什么问题？会执行哪些生命周期？
####能不能new一个Activity？
可以new，但是用他调用任何方法后，会报nullEcp,因为Android不同于java,他需要一个系统环境，直接new的话没有传入别的组件，比如说Context。同样的Service也不能new 

<b>Service</b>

介绍：Service没有界面，运行在后台

生命周期：
bandService()->onCreate->onBind->onUnbind->onDestroy

startService()->onCreate->onStart->onDestory

####Service是如何被创建的？
####Service多次启动同一个Service会发生什么？
首次启动
startService：onCreate->onStartCommand
bindService:onCreate->onBind
再次启动：
startService：onStartCommand
bindService:不执行生命周期
####Service能不能new？
也可以new，对象嘛，但是没用，他只能通过intent启动，new出来直接调用方法也会报错。
####Service的启动方式？
#####1 startService(intent service);

#####2 bindService(Intent service, ServiceConnection conn,int flags);
BIND_AUTO_CREATE
BIND_DEBUG_UNBIND
BIND_NOT_FOREGROUND

<b>BroadcastReceiver</b>

介绍：广播，接收系统发出的广播

生命周期：receiver

<b>ContentProvider</b>

介绍：内容提供者，可用来查询系统数据
###2. Activity与Service是如何通讯的？
####2.1 启动时Intent传值
####2.2 通过ServiceConnection IBinder交互
####2.3 接口回调
###3. Activity的启动方式有哪几种，大致介绍一下？
####3.1 standard
普通的启动方式，正常插入移除Activity栈
####3.2 singleTop
单一栈顶模式，若栈顶存在相同Activity,不会重新创建，而是直接唤醒栈顶Activity的onNewintent
####3.3 singleTask
只有一个实例，启动同一个Activity,会干掉该Activity栈之上的Activity，并将该置顶
####3.4 singleInstace
只有一个实例，单独运行在一个Acticity栈中，不允许有别的存在

###4. 做过哪些行业的项目，最出色的是哪一个？使用到了哪些技术？
###5. 到目前解决过的最麻烦的问题是什么？解决方法是什么？
###6. 介绍一下Android MediaSession播放框架
Android的媒体会话框架，可在Android内部将框架分为受控端和控制端，两端通过SessionToken来配对通讯。
###7. 有没有做过机顶盒的应用？焦点控制问题是怎么解决的？

###8. 如果要做一个音乐播放器，你的实现思路是什么？

##面试
面试中只有短短1~2小时，甚至更短，要在这些时间内体现出应聘者的价值，主要体现在

1. 开发能力是否能够胜任目前的工作
2. 沟通能力能否适应公司成员间正常交流
3. 学习能力与忠诚是否值得公司花费精力培养
4. 潜在能力能否为公司以后发展带来帮助

要通过介绍自己，表现出自己能够为公司产生价值，公司才会愿意雇佣你。 