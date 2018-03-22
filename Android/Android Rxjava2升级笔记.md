#Rxjava2 升级(踩坑)笔记

##前言
最近接触到别人代码的时候看到他们RxJava写的和我的不一样。Single、Completable、Disposable 什么不知道，CompositeDisposable这又是什么鬼？？doAfterTerminate()这个方法好可以再事件结束的时候调，但是我的代码里没这个方法！最终发现，Rxjava升级了。本文也只对新出现的变化做记录，没有rxjava基础的请自行学习。

感叹完别人这么用真是方便的同时，也开始着手做自己的库Rxjava的升级工作。当然遇到不少问题。

##正题
###遇到的问题

####DuplicateFileException: Duplicate files copied in APK META-INF/rxjava.properties

原因：Rxjava版本不统一，导致META-INF/rxjava.properties 这个合并冲突。不用去试图统一他(我至少没有成功)因为Retrofit、Rxjava、RxjavaAdapter 总有哪个里面的版本不一致。

解决办法：
在 app/build.gradle 中添加下面这段指令，意思是忽略这个打包冲突 
```
packagingOptions {
        exclude 'META-INF/rxjava.properties'
    }
```

#### 包名变了，类丢失
升级之后你会发现，Rxjava整个包名都TM变了！！对你没看错，同样的库，包名变了！而且不仅这样，直接删掉了Subscribe这些类，把他们改成的接口。所以你很多类要重新引用。并且可能有些方法会失效了。

<b>*所以老项目轻易不要升级！！！*</b>

```
    compile "io.reactivex.rxjava2:rxandroid:2.0.1"
    compile "io.reactivex.rxjava2:rxjava:2.1.0"
    compile "com.trello:rxlifecycle:0.3.1"
    compile "com.trello:rxlifecycle-components:0.3.1"
    compile "com.jakewharton.rxbinding:rxbinding:0.4.0"

    compile 'com.jakewharton.retrofit:retrofit2-rxjava2-adapter:1.0.0'
    compile "com.squareup.retrofit2:retrofit:2.2.0"
    compile "com.squareup.retrofit2:converter-gson:2.2.0"
```

这是我项目里面目前用到的Rx库。注意一点是RxJava升级之后Retrofit 最好也要同步升下级。

*注意以前的引用是這個：io.reactivex:rxjava:2.0.1*

*這個是新的：io.reactivex.rxjava2:rxjava:2.1.0*

###学习笔记

升级之后看到一些方法一脸懵逼，开始上网找教程。推荐一篇大家可以看下，比较浅显易懂。
https://www.jianshu.com/p/464fa025229e，一共有九篇，不过似乎没出全，不过作者的思路简单易懂，适合上手。

一口气撸完了九篇，简单实践 and 记录一下~

####Rxjava基础
#####创建三部曲
```
 private void demo01() {
        //上游被观察者---->发射数据
        Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                e.onNext("1111");
                e.onNext("2222");
                e.onNext("3333");
                e.onComplete();
            }
        });
        //下游观察者---->接收数据
        Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d("picher","onSubscribe");
            }

            @Override
            public void onNext(String s) {
                Log.d("picher","onNext："+s);

            }

            @Override
            public void onError(Throwable e) {
                Log.d("picher","onError");
            }

            @Override
            public void onComplete() {
                Log.d("picher","onComplete");
            }
        };
        //订阅
        observable.subscribe(observer);
    }
```

####Rxjava2 新出现的类

两个新东西：```ObservableEmitter``` 和 ```Disposable```

<b>*ObservableEmitter*</b>

1. 可以通过onNext() 发送数据
2. 调用onComplete() 或者 onError()将不在继续发送后续数据
3. onComplete()、onError() 只能同时调用一个

<b>*Disposable*</b>

可以理解为控制器，dispose()之后 下游将不在继续接收数据，但上游可以继续发送数据

<b>*Consumer*</b>

Rxjava1的版本 subscribe() 可以写 new fun1() fun2()... 但是rxjava2里面移除了这些，而用Consumer这个类代替，So！！！之前的如果你用挂了太多action或者fun,呵呵...改疯掉
 

####背压问题 (MissBackgroundException)

其实这问题Rxjava1也有，只是那会没有去了解是咋回事的，现在补上~

根本问题：下游处理能力不够，上游数据被放入容器等待处理，如果无限增长则内存会溢出OOM

- 同一个线程不会有这个问题，因为上游发送一下，下游处理一个

```
  //同一线程没问题
    private void demo02() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                for(int i=0; ;i++){
                    e.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d("picher",""+integer);
            }
        });
    }
```

- 异步任务的时候可能会出现背压问题
```
  //背压问题
    private void demo02() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                for(int i=0; ;i++){
                    e.onNext(i);
                }
                e.onComplete();
            }
        }).subscribeOn(Schedulers.io()).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d("picher",""+integer);
            }
        });
    }
```

#####背压问题的处理

<b>方案一：降低发送事件的次数</b>

我们知道背压问题是上游发的太快了，下游来不及处理导致挤压，所以我们可以减少发送的次数。比如加上Filter
过滤掉一部分可以过滤的数据。比如我们加上只让能被10整除的数据发送。

```
	//背压问题的处理--降低发送次数
    private void demo03() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                for(int i=0;;i++){
                    e.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io()).filter(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer % 10 == 0;
            }
        }).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d("picher",""+integer);
            }
        });
    }
```

<b>方案二：降低发送速度，让下游来得及去处理事件</b>

我们在每次发送事件之后睡2s,也可以解决问题。

```
 Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                for(int i=0;;i++){
                    e.onNext(i);
                    Thread.sleep(2000);
                }
            }
        })
```

<b>方案三：使用Flowable</b>

1. 设置BackpressureStrategy 策略
2. Subscription.request()

也就是RxJava2帮你处理了速度和数量的问题，也顺便帮你换了一个容器的容量。具体的去看下Blog吧，我不能保证有博主说的好。https://www.jianshu.com/p/a75ecf461e02

记录一下如果以后需要写上游发送量比较大的代码，则使用下面的写法。

```
	Flowable.interval(1, TimeUnit.MICROSECONDS)
                .onBackpressureDrop()  //加上背压策略
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Long>() {
                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.d(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(Long.MAX_VALUE);
                    }

                    @Override
                    public void onNext(Long aLong) {
                        Log.d(TAG, "onNext: " + aLong);
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.w(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "onComplete");
                    }
                });

```

举个生活中的例子：
面包厂一小时生产100个面包，面包店一小时卖出去50个面包，剩下的50个放到仓库里面，生产速度比消费速度大，当仓库存满的时候在存就爆仓了。

举个实际项目的例子：
电商项目里面需要对商品列表做一些比较等耗时处理，比如有200件待处理的商品数据，处理耗时5s，那么当发送到第127个商品事件的时候，如果处理算法还未计算完，那么就会出现问题。


##总结

1. Rxjava2 包名变了，导包的时候注意导入'io.reactivex.'这个包里面的，当然最好不要同时有两个版本的代码
2. 去掉了 fun,action等方法，
3. 添加了ObservableEmitter用来发射数据，Disposable 可以终止任务
4. 背压问题可以使用Flowable解决