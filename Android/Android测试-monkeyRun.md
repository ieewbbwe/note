#概述
昨天做完了单元测试，又被要求跑monkey，默念客户最屌，再去查查资料开搞。

##什么是monkey测试？
猴子测试，因为猴子只会乱点，这个测试模拟的就是在屏上乱点，乱按按键输入，检测多久会有异常。自动化测试的一种。

老司机们自己去看文档：[https://developer.android.com/studio/test/monkey.html](https://developer.android.com/studio/test/monkey.html)

##有哪些使用方法？
先跑起来试试，连接一台Android设备，并输入如下指令，包名一定要是在这条设备上有的

```
adb shell monkey -p yourPackageName -v 500>c:\monkey.txt
```

意思是在设备上对包名为yourPackageName进行500次的事件发送，log级别为-v 级，并将结果放在c:\monkey.txt里

log级别分为三个级别，一般用-v-v-v能获取到更多的信息

- -v 简单的列出启动通知，运行结果
- -v -v 在上一级基础上增加了事件发送的记录
- -v -v -v 包含了几乎所有的测试信息

给张截图吧，真的很完善了，自己去看文档测试：

最后说一下我使用的语句：

```
adb shell monkey –p xxx.xxx.xxx –throttle 1000 –ignore-crashes – ignore-timeouts –ignore-security-exceptions –ignore-native-crashs –moitor-native-crashes –s 1000 –v –v –v 20000>c:\mokeyLog.txt
```
意思为：

1. 运行设备上包名为xxx.xxx.xxx的应用程序；
2. 如果发生奔溃，请求超时，权限安全这些问题，记录下来并不会暂停 monkey的测试;
4. 每1000ms为一个活动项；
5. 向设备发送20000个事件
6. 将结果记录在c:\monkeyLog.txt里面

##问题：

####1 Monkey测试运行之后，log停止在了 :Sending rotation degree=0, persist=false，不会继续发事件出来
解决：因为使用的是小米手机，他关闭了旋转控制，于是到这边之后事件就停止了，换一个手机在试下就OK
