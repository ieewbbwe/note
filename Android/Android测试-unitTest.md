#1 概述
去年的一个电商的项目做完了，已经上线了，客户回来找我们做单元测试，性能测试，安全性测试，一堆测试项。我的内心是崩溃的，项目都上线了，隔了几个月了，项目组都换了两批了。没办法，客户是上帝，于是研究了一下Android目前的一些测试规范，记录一下吧。
#2 为什么要做测试
理清代码逻辑，让程序更加健壮，方法变更后跑一遍测试，若结果正确则证明没有影响到别的方法，更快的排查错误。

测试分类：

1. 单元测试 测试开发者编写的一小段代码，通常是一个方法，验证其执行是否成功
2. 功能测试 也就是大家都知道测试报告，黑盒测试，主要是测试人员对程序的功能点进行的测试
3. 安全性测试 主要是网络传输安全，本地数据加密的安全方面的测试
4. 性能测试 滑动性能，UI性能，网络性能

#3 需要测试哪些东西
目标组件生命周期，异步任务，消息传递，功能执行，界面跳转，计算结果
#4 怎么去做测试
单元测试的目标函数分类：

1. 对有明确返回值的函数，验证其返回值是否符合我们预期
2. 没有返回值的函数，但是有状态变更的，验证传入值之后的状态变更是否符合预期
3. 没有返回值，也没有状态变更的，验证其行为是否被执行

测试框架：

1. AndroidTest 需要运行在Android环境上
2. Robolectric 可以直接运行在JVM上，速度也更快，可以直接由Jenkins周期性执行，无需准备Android环境
## 使用Robolectric做单元测试（有问题）
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

### Robolectric常用测试方法
网络连接测试
计算结果测试
响应动作测试
点击跳转，点击弹窗

### Robolectric的一些问题
问：robolectric与Robotium区别？

答:这是两种测试框架，Robotium依赖Android平台，robolectric不依赖

问：testCompile 与 AndroidTestCompile 区别？

答：testCompile作用在 unit tests (src/test) 
androidTestCompile作用在 test api (src/androidTest)

问题：java.lang.NullPointerException: parentLoader == null && !nullAllowed

解决方案，不要使用androidTestCompile，意味着你创建的Test类不要放在AndrodTestCase下面
 testCompile 'org.robolectric:robolectric:3.3.2'

问题：首次执行 Robolectric回加载很长时间

因为  Robolectric又去maven仓库下了一些依赖包，那个仓库地址是国外的所以很慢

解决方案，把下载地址改为国内的[加速Robolectric下载依赖库及原理剖析](http://www.jianshu.com/p/a01628c3ea16)

问题：No such manifest file: build\intermediates\bundles\debug\src\main\AndroidManifest.xml
解决：配置一下sdk，目前只支持到21 @Config(constants = BuildConfig.class, sdk = 21)

问题：java.lang.NoClassDefFoundError: com/amazon/device/messaging/ADMMessageReceiver

三方库找不到问题，项目大了用的库太多，还有maven进来的，解决方法是配置@Config{Libraries} 但是我没有试成功，应该是路径问题，没找到解决办法

## 使用 robotium 做单元测试
MD,Robolectric配了半天没配好，各种问题，最后一个三方库找不到的问题实在懒得搞了，以后再看。用robotium也符合要求。
很简单，依赖这个库
```
    androidTestCompile 'com.jayway.android.robotium:robotium-solo:5.6.2'
```
在需要测试的类上右键->goto->Create test
或者直接在androidTest下面创建测试类

```
package com.xxx.xxx;

import android.content.Intent;
import android.support.test.InstrumentationRegistry;
import android.support.test.rule.ActivityTestRule;
import android.support.test.runner.AndroidJUnit4;
import android.test.ActivityInstrumentationTestCase2;

import com.robotium.solo.Solo;

import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(AndroidJUnit4.class)
public class LoadingActivityTest extends ActivityInstrumentationTestCase2 {

    private Solo solo;

    @Rule
    public ActivityTestRule<LoadingActivity> mActivityRule = new ActivityTestRule<>(
            LoadingActivity.class,
            false,
            false);

    @SuppressWarnings("unchecked")
    public LoadingActivityTest() throws ClassNotFoundException {
        super(LoadingActivity.class);
    }

    @Before
    public void setUp() throws Exception {
        solo = new Solo(InstrumentationRegistry.getInstrumentation());
    }

    @After
    public void tearDown() throws Exception {
        if (mActivityRule.getActivity() != null) {
            mActivityRule.getActivity().finish();
        }
    }

    @Test
    public void testSplash() {
        Intent intent = new Intent("com.xx.xxx.LoadingActivity");
        mActivityRule.launchActivity(intent);
        solo.assertCurrentActivity("is launch loadActivity", LoadingActivity.class);
        Assert.assertTrue("LoadingActivity launch fail", solo.waitForActivity(LoadingActivity.class, 1500));
    }

}
```
直接右键run就可以了，绿条通过红条不通过。主要就是Solo这个类，它包含了很多的测试方法

###常用测试方法
<b>启动界面</b>

```
 	@Rule
    public ActivityTestRule<StoreActivity> mActivityRule = new ActivityTestRule<>(
            StoreActivity.class,
            false,
            false);

 	@Test
    public void launcherActivity() {
		Intent intent = new Intent("com.xxx.xxx.TestActivity");
        mActivityRule.launchActivity(intent);
	}
  
```

<b>点击</b>

首先在setUp中申明一个Solo对象。

```
 	@Before
    public void setUp() throws Exception {
        solo = new Solo(InstrumentationRegistry.getInstrumentation());
    }
```

```
solo.clickOnText(String str);
```

在当前可见页面上搜素字符传，并点击这块内容

```
solo.clickOnView(View v);
```

点击这个View，View必须在当前页面可见

```
solo.clickInList(int line,int listIndex,int id);
```

点击可见页面上第listIndex个listView的第line行，布局是id的控件

```
solo.clickInRecycle(int index)；
```

与上面的方法类似，只是这个是适用于界面上是RecycleView的

<b>断言</b>

```
solo.assertCurrentActivity(String str,Class activityClass);
```

断言当前的Activity是activityClass这个，不是则报错

```
Assert.assertTrue(boolean condition);
```

断言当前这个是正确的

```
solo.assertMemoryNotLow();
``` 

断言目前系统可用内存是否过低，内存空间足够则通过

```
solo.isCheckBoxChecked(int index);
``` 

判断checkBox是否处于被选中的状态，可以通过index和text两种方法定位

<b>其余方法</b>

```
solo.sleep(int time);
```

当前操作睡time的时间，单位是毫秒，和thread一样，一般用于在点击之后等待数据响应或界面跳转需要的时间

```
solo.waitForView(View v);
```

在这一步等待界面上的v被加载出来，默认超时时间是20s

```
solo.waitForLogMessage(String str);
```

等待控制台打印出str的log信息，默认超时20s，一般用于网络访问时间或一些耗时计算的等待

```
solo.goBack();
```

按下返回键

```
solo.searchText(String str);
```

判断当前的屏幕中是否能找到指定的text

#5 测试完成后有哪些产出物
## 生成测试报告
项目根目录执行gradlew clean createDebugCoverageReport

然后就是漫长的等待，项目开始清理，并运行所有的测试项。

因为是用的robotium，所以必须有一台Android设备是链接的，不然运行半天最后test case跑不起来。

然后你就看到界面在那自己各种跳各种点各种测，很爽啊。

给大家看下结果吧，	如下图：

