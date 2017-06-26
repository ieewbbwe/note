#RxJava学习笔记（三）自定义操作符

## 概述

这一章介绍Rxjava如何自定义操作符。

首先要明白的是我们为什么要自定义操作符？

1. rxjava提供的操作符无法解决问题
2. 为了保证rxjava的链式调用规则
3. 秉着钻研精神

如果你的操作符是被用于创造一个Observable，而不是变换或者响应一个Observable，使用 create( ) 方法，不要试图手动实现 Observable。另外，你可以按照下面的用法说明创建一个自定义的操作符。

如果你的操作符是用于Observable发射的单独的数据项，按照下面的说明做：Sequence Operators 。如果你的操作符是用于变换Observable发射的整个数据序列，按照这个说明做：Transformational Operators 。

提示： 在一个类似于Groovy的语言Xtend中，你可以以 extension methods 的方式实现你自己的操作符 ,不使用本文的方法，它们也可以链式调用。详情参见 RxJava and Xtend

**序列操作符**

下面的例子向你展示了怎样使用lift( )操作符将你的自定义操作符（在这个例子中是 myOperator）与标准的RxJava操作符（如ofType和map）一起使用：

	fooObservable = barObservable.ofType(Integer).map({it*2}).lift(new MyOperator<T>()).map({"transformed by myOperator: " + it});
下面这部分向你展示了你的操作符的脚手架形式，以便它能正确的与lift()搭配使用。

实现你的操作符

将你的自定义操作符定义为实现了 Operator 接口的一个公开类, 就像这样：

	public class MyOperator<T> implements Operator<T> {
 	 public MyOperator( /* any necessary params here */ ) {
    /* 这里添加必要的初始化代码 */
	  }

 	 @Override
 	 public Subscriber<? super T> call(final Subscriber<? super T> s) {
  	  return new Subscriber<t>(s) {
      @Override
      public void onCompleted() {
        /* 这里添加你自己的onCompleted行为，或者仅仅传递完成通知： */
        if(!s.isUnsubscribed()) {
          s.onCompleted();
        }
      }

      	@Override
      	public void onError(Throwable t) {
        	/* 这里添加你自己的onError行为, 或者仅仅传递错误通知：*/
        	if(!s.isUnsubscribed()) {
        	  s.onError(t);
        	}
      	}

      	@Override
     	 public void onNext(T item) {
        	/* 这个例子对结果的每一项执行排序操作，然后返回这个结果 */
        	if(!s.isUnsubscribed()) {
          	transformedItem = myOperatorTransformOperation	(item);
        	  s.onNext(transformedItem);
        	}
    	  }
    	};
  	}
	}


**变换操作符**

下面的例子向你展示了怎样使用 compose( ) 操作符将你得自定义操作符（在这个例子中，是一个名叫myTransformer的操作符，它将一个发射整数的Observable转换为发射字符串的）与标准的RxJava操作符（如ofType和map）一起使用：

	fooObservable = barObservable.ofType(Integer).map({it*2}).compose(new MyTransformer<Integer,String>()).map({"transformed by myOperator: " + it});

下面这部分向你展示了你的操作符的脚手架形式，以便它能正确的与compose()搭配使用。

实现你的变换器

将你的自定义操作符定义为实现了 Transformer 接口的一个公开类，就像这样：

	public class MyTransformer<Integer,String> implements Transformer<Integer,String> {
  	public MyTransformer( /* any necessary params here */ ) {
    /* 这里添加必要的初始化代码 */
  	}

  	@Override
 	 public Observable<String> call(Observable<Integer> source) {
    /* 
     * 这个简单的例子Transformer应用一个map操作，
     * 这个map操作将发射整数变换为发射整数的字符串表示。
     */
    	return source.map( new Func1<Integer,String>() {
      		@Override
 		     public String call(Integer t1) {
  			      return String.valueOf(t1);
  			    }
  		  	} );
  		}
	}

