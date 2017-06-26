#RxJava 学习笔记（二）操作符

##1 常见RxJava操作符介绍

Rxjava中的操作符提供了一种函数式编程的方式，这里列举一下个人感觉用的比较多的操作符。并列举一些可能用到的实例。本文适合于快速上手，熟悉RxJava常见操作符的使用

###1.1 创建操作符

#### 1）Create

通过调用观察者的方法从头创建一个Observable。这个没啥好说的，最基本的一个。但是2.0之后好像有点变动，以后再看。

#### 2） From

将其它的对象或数据结构转换为Observable,会发射转入Iterable的每一项数据。个人理解就类似于for循环

#### 3） Just

将对象或者对象集合转换为一个会发射这些对象的Observable

#### 4） Timer

创建在一个指定的延迟之后发射单个数据的Observable，例如定时器

#### 5） Interval

创建一个定时发射整数序列的Observable，计时器，这里注意如果需要停止他需要在filter中用条件停止，或者unsubscribe()

###1.2 变换操作符

变换是RXjava中最有亮点的地方

#### 1) map

映射，通过对序列的每一项都应用一个函数变换Observable发射的数据，实质是对序列中的每一项执行一个函数，函数的参数就是这个数据项。

简单的可以理解为一对一的变换，输入一个对象A，输出一个对象B。

例如，打印一个班所有同学任意一门课程成绩。

	//打印每一名学生任意一门课程成绩
    private void demo11_2_3() {
        Observable.from(getStudent())
                .map(new Func1<StudentBean, Cause>() {
                    @Override
                    public Cause call(StudentBean studentBean) {
                        return studentBean.getCauseList().get(generateRandom(0, studentBean.getCauseList().size()));
                    }
                }).subscribe(new Action1<Cause>() {
            @Override
            public void call(Cause cause) {
                print("demo11_2_3", formatCause(cause));
            }
        });
    }

可以看到，在一条调用链上，我们通过from每次传入的是StudentBean,最终输出的是Cause，就是变换。

**变换原理分析**

激动人心的时候来了，来看一下map的源码。

类似代理机制。可以看到内部调用了一个lift(operate)的方法，看下他的实现（仅核心代码）

	// 这个是精简过后的代码，要看life的源码，可以去git仓库下载
	public final <R> Observable<R> lift(final Operator<? extends R, ? super T> operator) {
        return new Observable<R>(new OnSubscribe<R>() {
            @Override
            public void call(Subscriber<? super R> o) {
                 Subscriber<? super T> newSubscriber = hook.onLift(operator).call(o);
                 newSubscriber.onStart();
                 oldSubscriber.call(newSubscriber);
            }
        });
    }

这一部分没有搞清楚的去好好研究一下Rxjava基础，至少要明白订阅关系是如何建立的。

- 在oldObservable中创建了一个newObservable,他有一个newSubscriber；
- 在newObservable中call方法里，operator通过call方法生成一个newSubscriber,并且oldSubscriber通过call方法和newSubscriber建立关系
- 然后利用这个newSubscriber向Observable进行订阅

类似代理机制，拦截和处理事件序列之后的变换。我们还是举例来说明一下：

需求：Integer 转 String 这里不要说为什么不直接强制类型转换，举例子是为了辅助理解的，不要纠结。

做法一：利用map实现

	 private void demo11_2_4() {
        Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                subscriber.onNext(1);
            }
        }).map(new Func1<Integer, String>() {
            @Override
            public String call(Integer integer) {
                return integer + "";
            }
        }).subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                print("demo11_2_4", s);
            }
        });
    }

做法二：利用lift实现

	private void demo11_2_5() {
        Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("123");
            }
        }).lift(new Observable.Operator<Integer, String>() {
            @Override
            public Subscriber<? super String> call(final Subscriber<? super Integer> subscriber) {
                return new Subscriber<String>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(String s) {
                        int value = Integer.parseInt(s) * 2;
                        subscriber.onNext(value);
                    }
                };
            }
        }).subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                print("demo11_2_5", integer + "");
            }
        });
    }

做法三：代理

这里是把变换的代码用基本形式表示出来的


    private void demo11_2_6() {
	   final Observable.OnSubscribe oldOnSubScribe = new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                subscriber.onNext(1);
            }
        };
        // 创建一个被观察者
        Observable oldObservable = Observable.create(oldOnSubScribe);
        // 创建一个观察者
        final Subscriber oldSubscriber = new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String o) {
                print("demo11_2_6", o);
            }
        };	
  		//--------变换  用一个新的Observable处理之后代替发送订阅
    	Observable newObservable = Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                Subscriber newSubscribe = RxJavaPlugins.getInstance().getObservableExecutionHook()
                        .onLift(new OperatorMap<Integer, String>(new Func1<Integer, String>() {
                            @Override
                            public String call(Integer integer) {
                                return integer + "";
                            }
                        })).call(subscriber);
                newSubscribe.onStart();
                oldOnSubScribe.call(newSubscribe);
            }
        });
        //---------
		//订阅
        newObservable.subscribe(oldSubscriber);
	}

第三种方法是源码的实现方法，只是替换掉了Func、Action

![](http://ww4.sinaimg.cn/mw1024/52eb2279jw1f2rxcu9f46g20go0cz4qp.gif)

借用扔物线大神[《给 Android 开发者的 RxJava 详解》](http://gank.io/post/560e15be2dca930e00da1083)的一张图来说明会直观一点。

#### 2) FlatMap
 
扁平映射，将Observable发射的数据变换为Observables集合，然后将这
些Observable发射的数据平坦化的放进一个单独的Observable，可以认为是一个将嵌套的数据结构展开的过程。

简单的可以理解为一对多的变换，输入一个对象A，输出多个对象B，这是个人理解。实例，打印一个班每一名学生的每一门课程成绩。
	
	//打印每一名学生的每一门课程成绩
    private void demo11_2_2() {
        Observable.from(getStudent())
                .flatMap(new Func1<StudentBean, Observable<Cause>>() {
                    @Override
                    public Observable<Cause> call(StudentBean studentBean) {
                        print("demo11_2_2", studentBean.getName());
                        return Observable.from(studentBean.getCauseList());
                    }
                }).subscribe(new Action1<Cause>() {
            @Override
            public void call(Cause cause) {
                print("demo11_2_2", formatCause(cause));
            }
        });
    }

可以看到，我们通过from遍历了集合中的每一个学生，如果直接subscribe接收的会是StudentBean,但是我想要的数据又在StudentBean的cause集合里面，于是使用flatMap变换出一个Observable，在继续剩下的发射操作。

注意：使用Flatmap 发射的数据顺序会被打乱。要解决这个问题可以使用concatMap

####3) GroupBy

分组，将原来的Observable分拆为Observable集合，将原始Observable发射的数据按Key分组，每一个Observable发射一组不同的数据。

####4) Scan

扫描，对Observable发射的每一项数据应用一个函数，然后按顺序依次发射这些值，通常用来做一些通用公式的运算。

###1.3 过滤操作符

####1） Filter

过滤，过滤掉不符合要求的数据，只发射通过测试的。

####2） Distinct 

去重，过滤掉重复数据项。

####3） Debounce

只有在空闲了一段时间后才发射数据，通俗的说，就是如果一段时间没有操作，就执行一次操作，可以用来做搜索框。

实例：搜索框

	 // 一段时间没有变化则发送事件 0.5s 没有输入时则发送请求
    private void demo11_3_3() {
        RxTextView.textChanges(mDemoEt)//监听EditText变化
                .debounce(500, TimeUnit.MILLISECONDS)//设置时间
                .observeOn(AndroidSchedulers.mainThread())//不设置在主线程消费则toast缺少looper
                .filter(new Func1<CharSequence, Boolean>() {//过滤掉空字符串
                    @Override
                    public Boolean call(CharSequence charSequence) {
                        return charSequence.toString().trim().length() > 0;
                    }
                }).subscribe(new Action1<CharSequence>() {
            @Override
            public void call(CharSequence charSequence) {
                Log.d("demo11_3", charSequence.toString() + "thread:" + Thread.currentThread().getName());
                Toast.makeText(MainActivity.this, charSequence.toString(), Toast.LENGTH_SHORT).show();
            }
        }, new Action1<Throwable>() {//异常处理
            @Override
            public void call(Throwable throwable) {
                throwable.printStackTrace();
            }
        });
    }

#### 4） throttleFirst

取样，定期发射最新的数据，等于是数据抽样，可以用于防抖操作。

实例：防止按钮快速的多次点击 （例子中为了直观，设置的是5s响应一次点击事件

	//需求：防止按钮连续快速的点击
    private void demo11_3_4() {
        RxView.clicks(mDemoBt)//点击监听
                .throttleFirst(5000, TimeUnit.MILLISECONDS)//5s响应
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action1<Void>() {
                    @Override
                    public void call(Void aVoid) {
                        Log.d("demo11_3_4", "click");
                        Toast.makeText(MainActivity.this, "click", Toast.LENGTH_SHORT).show();
                    }
                });
    }

## 小结

这章主要介绍了一些RxJava中常见的一些操作符，和部分操作符的使用实例，重点分析了一下Rxjava中的变换原理。

当然Rxjava的操作符不止这些，还有oftype，delay等等很多不同功能的操作符，同时，我们也可以按照自己的需求来自定义操作符，下一章将介绍如何自定义操作符。

附上操作符文档地址：https://mcxiaoke.gitbooks.io/rxdocs/content/Operators.html

代码已经上传至GitHub：https://github.com/ieewbbwe/RxJavaSamples