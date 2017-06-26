#RxJava 学习笔记（一）概念
Rxjava 从去年开始就有耳闻，在各大开源项目中也可以看到有使用他的，各大牛也说他好用，既然大家都说好，具体好不好我们自己学着用一下就知道了，刚好这段时间项目闲下来就来学习一下RxJava。

###1 是什么
同样，在用一个东西之前，要先知道他是什么东西。

“RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences.”

这句话说得很全面了，RxJava是一个在Java虚拟机上的Reactive扩展程序：在Java虚拟机上使用可观测的序列来组成异步的，基于事件的程序的库。

简单来说：RxJava 是一个库，本质是异步

附上主页地址：https://github.com/ReactiveX/RxJava 

###2 可以做什么

####2.1 简化逻辑、异步操作

举个栗子，有若干名学生，现在想要打印出每一个学生的所有科目成绩。你可能会想到这样写：

      //for 循环
    private void demo7_1(List<StudentBean> students) {
        for (StudentBean studentBean : students) {
            Log.d("demo7_1", "姓名：" + studentBean.getName());
            for (Cause cause : studentBean.getCauseList()) {
                Log.d("demo7_1", formatCause(cause));
            }
        }
    }

    private String formatCause(Cause cause) {
        return String.format("caseName:%s->>causeGrade:%s", cause.getName(), cause.getGrade());
    }
	

这样当然OK，如果我还要加一个条件，打印出每一个男学生的所有科目成绩。你是不是又想用到一层循环了。在比如我学生量超大，你是不是还想到了不能在主线程操作了？写过大项目都知道迷之循环谜之缩进毁一生。逻辑不多的情况没有影响，一旦逻辑多起来你可能都不知道自己写的是什么。

我们再来看一下用RxJava如何解决这个问题

	 //RxJava
    private void demo7_2(List<StudentBean> students) {
        from(students).subscribe(new Action1<StudentBean>() {

            @Override
            public void call(StudentBean studentBean) {
                Log.d("demo7_2", "姓名：" + studentBean.getName());
                for (Cause cause : studentBean.getCauseList()) {
                    Log.d("demo7_2", formatCause(cause));
                }
            }
        });
    }
先简单说一下操作符，from可以把传入的集合依次遍历输出，后面还会专门讲操作符，不要急。action1是RxJava的一个接口，call(T param)，先简单理解为一个处理结果的回调。

当然你们会说不就用一个from代替了一层循环吗？并没有什么卵用，还不是内部有一个for循环，当然我们可以更优雅一点。

	private void demo7_3(List<StudentBean> students) {
        from(students).map(new Func1<StudentBean, List<Cause>>() {

            @Override
            public List<Cause> call(StudentBean studentBean) {
                Log.d("demo7_3", "姓名：" + studentBean.getName());
                return studentBean.getCauseList();
            }
        }).subscribe(new Action1<List<Cause>>() {

            @Override
            public void call(List<Cause> causes) {
                for (Cause cause : causes) {
                    Log.d("demo7_3", formatCause(cause));
                }
            }
        });
    }

有好事的（就是我）又说在用一个from不可以吗？反正都是遍历。当然可以，但是就不能保证一个流式了，map里原理其实也是这样，但是千万不要把map理解为又是一个from，后面单独有一章将变换。

在加上我们刚才说的逻辑，打印性别为男的，新开线程中运行。

	   private void demo7_4(List<StudentBean> students) {
        from(students)//遍历学生数组
                .subscribeOn(Schedulers.io())//在IO线程中运算
                .observeOn(AndroidSchedulers.mainThread())//在主线程中回调
                .filter(new Func1<StudentBean, Boolean>() {

                    @Override
                    public Boolean call(StudentBean studentBean) {
                        return studentBean.isMale();//筛选出性别为男的数据
                    }
                })
                .map(new Func1<StudentBean, List<Cause>>() {

                    @Override
                    public List<Cause> call(StudentBean studentBean) {
                        Log.d("demo7_4", "姓名：" + studentBean.getName() + "性别：" + (studentBean.isMale() ? "男" : "女"));
                        return studentBean.getCauseList();
                    }
                })
                .subscribe(new Action1<List<Cause>>() {

                    @Override
                    public void call(List<Cause> causes) {
                        for (Cause cause : causes) {
                            Log.d("demo7_4", formatCause(cause));
                        }
                    }
                });
    }

有人会说，这简单个屁啊，代码量变多了，我还看不懂了！您消消气，我开始也说了是逻辑的简化，在看一下注释，是不是都在一条调用链上一路“.”下来的，每一步做的什么一目了然是不？如果项目放在那大半年在突然让你去加个功能，这样看起来逻辑梳理的要快得多。

来解释一下：

**Scheduler 的API**

subscribeOn(Scheduler scheduler)

决定了subscribe() 之前的程序运行在哪个线程中，我们叫事件产生的线程，主要有：

1. io()指 I\O线程，用于一些读写操作，例如查找数据库，网络请求,内部是无上限的线程池;
2. computation() 计算线程，用于处理一些CPU密集计算，内部是核数的线程池;
3. newThread() 新起一个线程；
4. immediate() 当前线程，也是默认值;
5. AndroidSchedulers.mainThread()，Android主线程即UI线程，使用这个需要额外添加一个依赖

	    compile 'io.reactivex:rxandroid:1.0.1'

observeOn(Scheduler scheduler)

决定了subscribe()响应在哪个线程，我们叫事件消费的线程，通常用于做一些耗时操作之后更新UI的一些动作，就类似[Handler 消息机制](http://blog.csdn.net/lmj623565791/article/details/38377229/)，主要的Scheduler也和上面的一致。

又有好事的问了，能不能多切换几次，答案是可以！but，subscribeOn以第一次出现的为准；observeOn以最后一次出现的为准！

关于Scheduler的线程切换原理我们之后在说，在这之前还需要来了解一些其他的知识。

###3 怎么使用

看完刚那个例子我们已经大致了解到RxJava 的一些优势了，好就好在能把好多复杂的逻辑都穿成一条线，并且还能异步操作。

RxJava的API比较多，操作符也比较多，在学习API和操作符，在穿插一些原理分析，自然就知道怎么用了。

####3.1 API介绍及原理分析

**1） 概念** [观察者模式](https://www.baidu.com/link?url=wZI2-YySe5mipxWDkgbwW670-vOfr9DMqkMnSQM3V0Gf-W1-dfxdi1tkDeurteg5adj834Qv1HlODpJBLAbb_IKnIjwJmLSveDuRPkxDh_7&wd=&eqid=c80bbb3c000100840000000658413295)

链接里是鸿洋大神的透彻分析，我这里就简单说一下自己的学习理解。
还是举个栗子：虚拟机告诉代码“你在出异常的时候和我说一下”。

这个例子里面，"虚拟机"是观察者，"代码"是被观察者，"告诉"是关联关系在代码出错的时候虚拟机会做相应的动作。

那么我们知道观察者模式有两个实体一个关系：
1. 观察者 Observer
2. 被观察者 Observable
3. 订阅关系 subscribe
在举一个开发中的例子：

	   View.OnClickListener observer = new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                Log.d("webber","点击了！");
            }
        };
        mLoginBt.setOnClickListener(observer);

这里我故意把Listener起名为observer，方便理解,在这个例子中mLoginBt这个View是被观察者，observer这个监听器是观察者，他们通过setOnClickListener关联起来，observer就能在View被点击的时候收到通知。

这个概念抽象出来就是

Button(被观察者)；onClick(事件)；Listener(观察者)
![](http://ww4.sinaimg.cn/mw1024/52eb2279jw1f2rx42h1wgj20fz03rglt.jpg)
![](http://ww3.sinaimg.cn/mw1024/52eb2279jw1f2rx4446ldj20ga03p74h.jpg)
图片出自[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)，感谢扔物线大神。

**2）基本实现**

Ok，在了解基本概念之后我们要开始使用这个工具了！我们知道需要有观察者、被观察者、关联关系

1. 创建Observer

		//创建观察者
        Observer<String> observer = new Observer<String>() {

            @Override
            public void onCompleted() {
                Log.d("webber", "onCompleted:");
            }

            @Override
            public void onError(Throwable e) {
                Log.d("webber", "onError:" + e.getMessage());
            }

            @Override
            public void onNext(String s) {
                Log.d("webber", "onNext:" + s);
            }
        };

2. 创建Observable
	
	 //创建被观察者
        Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                //subscriber.onStart();
                Log.d("network", "call" + Thread.currentThread().getName());
                subscriber.onNext("item1");
                subscriber.onNext("item2");
                subscriber.onNext("item3");
                subscriber.onCompleted();
                //subscriber.onError(new Throwable("我出错了！！！"));
            }
        });


3. 关联关系

	 observable.subscribe(observable);

有些细心的可能会发现不是应该观察者订阅被观察者吗，这里为什么反了？
这就有点像是杂志订阅了读者。但是如果颠倒一下关系就会影响整个流式布局，得不偿失。

 

