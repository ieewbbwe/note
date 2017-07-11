#1 概述
去年的一个电商的项目做完了，已经上线了，客户回来找我们做单元测试，性能测试，安全性测试，一堆测试项。我的内心是崩溃的，项目都上线了，隔了几个月了，项目组都换了两批了。没办法，客户最牛逼，必须得做，于是研究了一下Android目前的一些测试规范，记录一下吧。
#2 为什么要做测试
1. 理清代码逻辑
2. 让程序更加健壮
测试分类：
1. 单元测试 测试开发者编写的一小段代码，通常是一个方法，验证其执行是否成功
2. 功能测试 也就是大家都知道测试报告，黑盒测试，主要是测试人员对程序的功能点进行的测试
3. 安全性测试 主要是网络传输安全，本地数据加密的安全方面的测试
4. 性能测试 滑动性能，UI性能，网络性能

*去他妈的，都是客户逼着我做的，功能都开发不完，我做个毛的测试。*
#3 需要测试哪些东西
目标组件生命周期，异步任务，消息传递，功能执行
#4 怎么去做测试
单元测试的目标函数分类：
1. 对有明确返回值的函数，验证其返回值是否符合我们预期
2. 没有返回值的函数，但是有状态变更的，验证传入值之后的状态变更是否符合预期
3. 没有返回值，也没有状态变更的，验证其行为是否被执行

测试框架：
1. AndroidTest 需要运行在Android环境上
2. Robolectric 可以直接运行在JVM上，速度也更快，可以直接由Jenkins周期性执行，无需准备Android环境
## 使用Robolectric做单元测试
Robolectric是一个Android单元测试框架。

可以测试Activity的跳转、Activity展示View（包括菜单）和Fragment到View的点击触摸以及事件响应，同时Robolectric也能测试Toast和Dialog。对于需要网络请求数据的测试，Robolectric可以模拟网络请求的response。对于一些Robolectric不能测试的对象，比如ConcurrentTask，可以通过自定义Shadow的方式现实测试。
首先添加依赖
```
testCompile 'junit:junit:4.10'
testCompile 'org.robolectric:robolectric:3.0'
```
Mock配置如果要测试的目标对象依赖关系较多，需要解除依赖关系，以免测试用例过于复杂，用Robolectric的Shadow是个办法，但是推荐更加简单的Mock框架，比如Mockito，该框架可以模拟出对象来，而且本身提供了一些验证函数执行的功能。Mockito配置如下：
```
repositories {
    jcenter()
}
dependencies {
	//don't use androidTestCompile here!!!
    testCompile "org.mockito:mockito-core:1.+"
}
```



问：robolectric与Robotium区别？
问：testCompile 与 AndroidTestCompile 区别？
testCompile作用在 unit tests (src/test) 
androidTestCompile作用在 test api (src/androidTest)

问题：java.lang.NullPointerException: parentLoader == null && !nullAllowed
不要使用androidTestCompile，意味着你创建的Test类不要放在AndrodTestCase下面
 testCompile 'org.robolectric:robolectric:3.3.2' 

#5 测试完成后有哪些产出物
## 生成测试报告
