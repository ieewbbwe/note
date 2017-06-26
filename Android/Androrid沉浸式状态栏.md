#Android 沉浸式状态栏

##1 是什么
Google推出Android 5.0 之后，带来了一波Material Design的设计风，旗下许多产品都转成了这种设计风格，今天来总结一下现在Android应用中一个常见的效果，沉浸式状态栏。
（[为什么在国内会有很多用户把「透明栏」（Translucent Bars）称作 「沉浸式顶栏」？](https://www.zhihu.com/question/27040217)）

这种沉浸式的体验主要是在4.4以上的版本中支持的，但是4.4和5.0以上效果略有不同，下面先看一下效果：

- 4.4 模拟器上的效果
- 5.0 模拟器上的效果

##2 怎么做

翻阅了大量的资料，发现实现沉浸式状态栏的方式有许多种，其主要思路都差不多1 空出状态栏的位置 2 填充一个背景
下面介绍一下具体怎么做：

1. 最简单的方式

使用Android studio开发的兄弟在创建项目的时候，如果有大于21的sdk那系统会自动构建一个支持沉浸式的项目，当然也可以自己实现，很简单，自定义一个主题，继承自AppCompat下的主题即可。

```
 <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```


