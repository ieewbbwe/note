# Android 8.0 升级笔记

## 前言
Google 再2017年就发布了Android 8.0,并且强制AppStore上得应用都要升级，国内得不晓得。为了防止出现之前升级6.0 得时候权限问题导致Crash这种情况得发生...这次很小心得去看了Google得升级意见，小伙伴们可以自行去看(https://developer.android.com/index.html)。
我大致记录以下有以下几点需要注意的:

- 升级API版本号到26
- 移除隐式意图接收器
- 更新应用图标 

## 正题

### Android studio 3.0.1 下载
提前说下Google 建议使用Android studio 3.0 以上得版本，给一个地址需要得自己去下载吧。

官方（需要翻墙）:https://developer.android.com/studio/index.html
国内：http://www.android-studio.org/ 

### 编译版本&相关库升级

<b>找到你项目的build.gradle,主要关注以下几个,升级到26</b>

```
android {
    compileSdkVersion 26
    buildToolsVersion "26.0.2"
    defaultConfig {
        minSdkVersion 16
        targetSdkVersion 26
    }
}
```

<b>升级相关的支持库</b>
如果你不这么做studio会提示警告，但是也不会影响运行，如果要用库里最新的控件还是更新下吧。代码洁癖的不会想看到大段的黄色警告和红色波浪线...
```
 compile 'com.android.support:appcompat-v7:26.1.0'
    compile 'com.android.support:recyclerview-v7:26.1.0'
    compile 'com.android.support:cardview-v7:26.1.0'
    compile 'com.android.support:design:26.1.0'
```

我暂时就用到这些，有些项目里有用到Google Service的服务也要升级到 11.8.0

注意一点是studio 3.0以下的版本会提示找不到 "26.1.0"的库，而且你在Sdk Manager也找不到，需要再project的build.gradle加google的库，jCneter 里没有。

```
allprojects {
    repositories {
        jcenter()
        google()
    }
}
```
或者这样写

```
maven {
            url 'https://maven.google.com/'
            name 'Google'
        }
```

当然如果是Studio3.0 以上的版本IDE 会自动帮你加上！

### 应用隐式意图检查

Google为了优化(填以前的坑)系统性能，让应用无法再Manifast注册隐式意图了，因为Google认为这样做太消耗资源，特么的开发者写的APP消耗资源，用户觉得性能差，出了事要Google的Android系统背锅！他们不干了，要开始约束开发者行为。

举个例子：

一款社交图片APP，开发者希望再每次链接数据线的时候备份照片并清理缓存。

那么他需要写一个广播接收器，并再manifest中注册这个接收器，接收ACTION_POWER_CONNECTED 的广播。

```
<receiver android:name="com.xxx.PowerConnectReceiver">
            <intent-filter>
                <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
            </intent-filter>
        </receiver>
```

再Android 8.0 上Google说你不能这么干！这会在每次链接电源的时候去通知所有注册了这个广播的应用浪费系统资源！所以移除吧。

当然也不是所有的意图都被过滤掉了，官网也说了有些意图不会受这条约束(https://developer.android.com/guide/components/broadcast-exceptions.html)，我从官网摘录了下来，访问不了外网的可以搜以下你的应用有下面这些隐式意图的话就不用做8.0的适配，相反你就需要做适配


```
ACTION_LOCKED_BOOT_COMPLETED, ACTION_BOOT_COMPLETED
由于这些广播只能在第一次启动时只发送一次，因此许多应用程序需要接收此广播来安排作业，警报等等。

ACTION_USER_INITIALIZE, "android.intent.action.USER_ADDED", "android.intent.action.USER_REMOVED"
这些广播受到特权的保护，因此大多数普通应用程序无法收到它们。

"android.intent.action.TIME_SET", ACTION_TIMEZONE_CHANGED, ACTION_NEXT_ALARM_CLOCK_CHANGED
时钟应用程序可能需要接收这些广播，以在时间，时区或警报发生更改时更新警报。

ACTION_LOCALE_CHANGED
只有在语言环境发生变化时才会发送，这种情况并不常见。应用程序可能需要在区域设置更改时更新其数据。

ACTION_USB_ACCESSORY_ATTACHED, ACTION_USB_ACCESSORY_DETACHED, ACTION_USB_DEVICE_ATTACHED, ACTION_USB_DEVICE_DETACHED
如果一个应用程序需要了解这些USB相关事件，目前还没有一个好的选择来注册广播。

ACTION_CONNECTION_STATE_CHANGED, ACTION_CONNECTION_STATE_CHANGED, ACTION_ACL_CONNECTED, ACTION_ACL_DISCONNECTED
如果应用程序接收到这些蓝牙事件的广播，用户体验不太可能受到影响。

ACTION_CARRIER_CONFIG_CHANGED, TelephonyIntents.ACTION_*_SUBSCRIPTION_CHANGED, "TelephonyIntents.SECRET_CODE_ACTION"
OEM电话应用程序可能需要接收这些广播。

LOGIN_ACCOUNTS_CHANGED_ACTION
某些应用程序需要知道登录帐户的更改，以便他们可以为新帐户和更改的帐户设置预定的操作。

ACTION_PACKAGE_DATA_CLEARED
只有在用户从“设置”中明确清除其数据时才会发送，因此广播接收器不太可能会显着影响用户体验。

ACTION_PACKAGE_FULLY_REMOVED
有些应用程序可能需要更新其存储的数据时，另一个包被删除; 对于这些应用程序，没有好的选择注册这个广播。

注意：其他与包相关的广播（例如ACTION_PACKAGE_REPLACED）不受新限制的限制。这些广播已经足够普遍，对于免除这些广播有潜在的性能影响。

ACTION_NEW_OUTGOING_CALL
针对拨打电话的用户采取行动的应用程序需要接收此广播。

ACTION_DEVICE_OWNER_CHANGED
这个广播不经常发送; 一些应用程序需要接收它，以便他们知道设备的安全状态已经改变。

ACTION_EVENT_REMINDER
由日历提供程序发送，以便将事件提醒发送到日历应用程序。由于日历提供程序不知道日历应用程序是什么，这个广播必须是隐含的。

ACTION_MEDIA_MOUNTED, ACTION_MEDIA_CHECKING, ACTION_MEDIA_UNMOUNTED, ACTION_MEDIA_EJECT, ACTION_MEDIA_UNMOUNTABLE, ACTION_MEDIA_REMOVED, ACTION_MEDIA_BAD_REMOVAL
这些广播是由于用户与设备的物理交互（安装或移除存储卷）或作为引导初始化的一部分（因为可用卷被挂载）而发送的，因此它们不是常见的并且通常在用户的控制之下。

SMS_RECEIVED_ACTION, WAP_PUSH_RECEIVED_ACTION
这些广播是由SMS收件人应用程序依靠。
```

*注：需要签名权限的广播不受此限制所限*

那么想实现上面那个功能该怎么办呢？我就是想在应用中接收连接电源的广播要怎么做呢？

- 让用户再Setting 中自己打开(基本不可取)
- 在应用启动的时候动态的注册广播
	```
		Context.registerReceiver() 
	```
- 使用JobScheduler

简单一句介绍，5.0 之后系统会干掉轮询等保活机制，JobScheduler系统不会杀。

有兴趣的哥们自己去研究下，不难理解。

#### 显示意图 & 隐式意图

举个例子，如果忘记了显示\隐式意图的可以看下。

##### 显式意图

显式意图：调用Intent.setComponent()或Intent.setClass()方法明确指定了组件名的Intent为显式意图，显式意图明确指定了Intent应该传递给哪个组件。

```
Intent intent = new Intent(context,AnotherActivity.class);//第二个参数是待启动Activity的类的反射  
intent.putExtra("name",name);//可通过意图向另一个Activity传递数据  
startActivity(intent);//跳转到另一个Activity  
```

##### 隐式意图

隐式意图：没有明确指定组件名的Intent为隐式意图。 Android系统会根据隐式意图中设置的动作(action)、类别(category)、数据（URI和数据类型）找到最合适的组件来处理这个意图。

```
<intent-filter>  
   <action android:name="android.intent.yinsiyitu.action"/>  
   <category android:name="android.intent.category.DEFAULT"/>  
   <data android:mimeType="application/person"/>  
   <data android:scheme="jianren" android:host="www.ggl.com"/>  
</intent-filter>  
```

或者在Activity中写

```
Intent intent = new Intent();  
intent.setAction("android:intent.yinsiyitu.action");  
intent.addCategory(Intent.CATEGORY_DEFAULT);  
//intent.setData(Uri.parse(jianren://www.ggl.com));//会清除前面所有set的type  
//intent.setType("application/person");//会清除前面所有的set的data  
//这是setData和setType两全的方法，另外如果上面的Activity定义了host，则这里一定也要指定  
intent.setDataAndType(Uri.parse("jianren://www.ggl.com"),"application/person");  
//如果上面的Activity没有定义host，则Uri.parse("jianren:");至少要写到冒号，不可以只写Uri.parse("jianren")  
startActivity(intent);  
```

### adative图片适配

https://developer.android.com/guide/practices/ui_guidelines/icon_design_adaptive.html

去看下吧，我觉得我讲的不会有这个细了，我大致说下流程。
1. 准备background、foreground icon

用人话说就是让UI给图，把logi拆成ICON和背景色 两张图

2.  创建/res/mipmap-anydpi/ic_launcher.xml

```
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
	<!-- Logo背景色 -->
    <background android:drawable="@color/ic_background"/>
	<!--Logo 图标-->
    <foreground android:drawable="@mipmap/ic_foreground_layer"/>
</adaptive-icon>
```

注：如果不做这个适配的话8.0 的设备上你的App 图标可能是很丑的一个被圆形包住方块

### 其他一些API变动

官网上摘录下来的。

<table broder='0'>
  <tbody><tr>
    <th>变化</th>
    <th>摘要</th>
    <th>其他参考资料</th>
  </tr>
  <tr>
    <td>
      隐私性
    </td>
    <td>
      Android 8.0 不支持使用 net.dns1、net.dns2、net.dns3 或 net.dns4 系统属性。
    </td>
    <td>
      <a href="https://developer.android.com/preview/behavior-changes.html#pr">行为变更：隐私性</a>
    </td>
    </tr>
    <tr>
    <td>
      实行了可写且可执行的代码段
    </td>
    <td>
    对于原生库，Android 8.0 实行的规则是：数据不应可执行，代码不应可写。
    </td>
    <td>
      <a href="https://developer.android.com/preview/behavior-changes.html#ndk">行为变更：原生库</a>
    </td>
  </tr>
  <tr>
    <td>
      ELF 标头和节验证
    </td>
    <td>
      动态链接器对 ELF 标头和节头中的更多值进行检查，如果值无效则失败。
    </td>
    <td>
      <a href="https://developer.android.com/preview/behavior-changes.html#ndk">行为变更：原生库</a>
    </td>
  </tr>
<tr>
  <td>
    通知
  </td>
  <td>
    以 SDK 的 Android 8.0 版本为目标平台的应用必须实现一个或多个通知渠道，以便向用户发布通知。
</td>
<td>
      <a href="https://developer.android.com/preview/api-overview.html#notifications">API 概览：通知</a>
  </td>
</tr>
<tr>
    <td>
      <code><a href="https://developer.android.com/reference/java/util/List.html#sort(java.util.Comparator&lt;? super E&gt;)">List.sort()</a></code> 方法
    </td>
    <td>
  该方法的实现不得再调用 <code><a href="https://developer.android.com/reference/java/util/Collections.html#sort(java.util.List&lt;T&gt;)">Collections.sort()</a></code>，否则应用将因堆栈溢出而引发异常。
    </td>
    <td>
      <a href="https://developer.android.com/preview/behavior-changes.html#o-ch">行为变更：集合的处理</a>
    </td>
  </tr>
  <tr>
    <td>
      <code><a href="https://developer.android.com/reference/java/util/Collections.html#sort(java.util.List&lt;T&gt;)">Collections.sort()</a></code> 方法
    </td>
    <td>
        在列表实现中，<code><a href="https://developer.android.com/reference/java/util/Collections.html#sort(java.util.List&lt;T&gt;)">Collections.sort()</a></code> 现在会引发 <code><a href="https://developer.android.com/reference/java/util/ConcurrentModificationException.html">ConcurrentModificationException</a></code>。
      </td>
      <td>
        <a href="https://developer.android.com/preview/behavior-changes.html#o-ch">行为变更：集合的处理</a>
      </td>
  </tr>
</tbody></table>


## 总结

比较重要的几个就是移除隐式意图，更换app logo图，升级相关库；如果有用到API 在上面有说明改变的也要注意以下，目前来看还没有遇到别的坑。发现来再来补上！
