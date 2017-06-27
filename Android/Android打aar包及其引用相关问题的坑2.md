
## 概述

昨天成功制作了一个mavenLocal，成功把sdk给了客户，但是由于我们lib工程比较杂，客户的项目也比较杂，就出现了很多问题，今天解决完这一大票问题后，抽个空来总结一下。主要就是apk的打包，资源合并问题。

### Android清单文件合并

问题一 "manifest merger failed with multiple errors" 

清单文件的合并问题，主要有以下几种常见的：

<b>1 android:icon和android:theme 冲突，库和你的工程都有这个配置</b>

解决办法：在Manifest.xml的application标签下添加tools:replace="android:icon, android:theme"（多个属性用,隔开，并且记住在manifest根标签上加入xmlns:tools="http://schemas.android.com/tools"，否则会找不到namespace）

<b>2 sdk版本不一致</b>

解决办法：最好让库中的sdk版本和项目中的一致。如果有不一致的情况，项目的minSdkVersion要低于库里面的minSdkVersion

如果不好配置，可以在项目的manifest使用
```<uses-sdk android:minSdkVersion="15">```
来强制设定最小sdk，但是这样做对于要求高的库会报错，

"Suggestion: use tools:overrideLibrary="" to force usage"

这时可以<uses-sdk tools:overrideLibrary="xxx.xxx.xxx"/>，意思是项目中的AndroidManifest.xml和第三方库AndroidManifest.xml合并时可以忽略最低版本限制。负面影响还没有看出来。

我的项目里面就是这个坑！我之前打的aar里面minSdk是16，客户配置的是15，就会编译报错。

参考资料：
[清单文件合并(译)](http://blog.csdn.net/maosidiaoxian/article/details/42671999)
，[Android官方的清单合并文档](https://developer.android.com/studio/build/manifest-merge.html)

### 资源文件合并

"Could not expand ZIP"

由于Android打包apk的时候会合并库里面的资源文件，有重复的资源，重复的jar，就会报错。

报这个错误的可能情况有一下几种：

<b>1 Android jar包冲突</b>

项目中多处引用同一个jar，会出现该问题。我项目中的问题是："duplicate entry 'okhttp3/Address.class"
okhttp3被重复引用了。

解决办法：

1 改变库为provided模式

Provided是对所有的build type以及favlors只在编译时使用，类似eclipse中的external-libs,只参与编译，不打包到最终apk。

第一步 去掉```compile fileTree(dir: 'libs', include: ['*.jar'])```

第二步 将libs中的jar以```compile files('libs/gson-2.4.jar')```的形式包含进来

第三部 将冲突的库改为```provided files('libs/okhttp-3.2.0.jar')```

这个解决办法的思路是：在编译的时候各自库使用各自库的jar文件，在最终编译打包的时候，不会打包进重复的jar。

注意：使用Provided 的前提是你在最终打包的时候必须要存在一个该jar被打包进apk，不然运行的时候会找不到类。例如上例，okhttp在我自己的库中是provided，但是在客户那边的库里面就必须要有一个compile的引用

2 编译库文件时不包含冲突的jar

有一些库中使用到其他的库或者jar，就会有冲突。例如我项目中

"duplicate entry org/slf4j.class"

这时双击shift，查一下slf4j是哪些库在使用，如下图，发现是ormlit在使用，所以我们在
```
compile files('libs/ormlite-android-4.48.jar')
	    {exclude group: 'org.slf4j'}
```
不要把它包含进来就可以了。

参考资料：[Android studio 解决重复依赖问题](http://blog.csdn.net/cx1229/article/details/52786168)

3 查找冲突的jar，并删掉无用jar

按照上面那个方法找到重复jar的位置后，如果这个库你没有用到就删了吧。也再次强调，框架，库一定要简洁，不要他臃肿了。

4 改变jar的依赖方式为```dependencies {compile 'com.squareup.okhttp:okhttp:2.7.5'}```

只是在stackoverFlow上面看到有人说这么做，还没有去试，因为客户那边基本都是jar导入的。以后新建工程的时候尽量使用compile吧。

<b>2 attr属性申明重复</b>

"Attribute ＂xxx＂ has already been defined"

比如库中attr.xml 中定义了 
```<attr name="scrollingCache" format="boolean"/>```;
同时其他module中也定义了scrollingCache这个属性，那么就会报这个错误。

解决办法：如果没用到的话删掉重复的属性值，否则改属性值的名字。

我项目中是AblListView 中的listSelector、drawSelectorOnTop...这些属性冲突，客户的库里也有这个控件。我是改名解决的，但是重复的值多了就不能这么搞了，其他办法还有找到。

其实看起来就是一个apk打包的问题，但是由于库复杂了，解决一个问题又来一个问题，搞得头都大了。总结一下：

1. 框架不要过于臃肿，无用的lib在框架搭建的时候要剔除掉
2. 项目中对库的直接依赖不要过多
3. 项目中的库尽量使用compile的方式导入
4. 库与项目的sdk版本尽量统一
5. 库与项目的一些依赖，例如design，suport版本尽量统一，可以在gradle中申明ext来做库版本管理

就这么多了，希望能为大家提供到帮助。我去精简框架去了...
 




