##1 环境搭建
###1.1 IDE选择
<b>Sublime. Text 支持的插件多，前端的猿应该很熟</b>
<br/>下载地址：[http://www.sublimetext.com/3](http://www.sublimetext.com/3)
<br><b>Atom. 插件很多，支持不错</b>
<br/>下载地址：[https://atom.io/](https://atom.io/)
<br><b>WebStorm. JetBrains 全家桶，自动下载库很方便</b>
<br/>下载地址：[https://www.jetbrains.com/webstorm/](https://www.jetbrains.com/webstorm/)

我用的是WebStorm，因为我做Android开发的用的Android Studio和这个IDE有很多相似的地方，上手快啊，而且自动下载一些包也挺方便的。(收费自己找激活码或者服务器吧很简单)

其他的自行了解：[十大最受欢迎的 React Native 应用开发编辑器](https://blog.csdn.net/csdnnews/article/details/78042121)

###1.2 运行环境配置
1. 安装node环境，[https://nodejs.org/en/](https://nodejs.org/en/)点击下载V6.xxx安装即可；<br/>用命令行 node -v 检验是否安装成功 
2. 安装React Native.<br> npm install -g react-native-cli<br>*NPM 一个包管理器，安装完node之后会自动被安装*
3. 安装Android\IOS环境，自已找下吧

##2 项目运行
###2.1 使用WebStorm创建并以Android运行一个项目。
1. 打开WebStorm 如图创建项目，之后IDE会自动去下载相关资源包，配置项目，第一次有点慢不用管它。
2. 项目初始化完成以后，看右上角选择Android 再点Run，就能跑到虚拟机上了。
我这里用的是AS自带的虚拟机，当然也可以用GenyMotion、夜神，或者真机。
OK，跑上去了，看下效果图，看到的效果就写在App.js里面。

几个亮点：

1. 如果只更新JS代码而没有动资源文件那么可以直接ReloadJs，不用再跑一遍
2. 可以使用Chrom协助调试JS代码

<b>如何使用Chrom 调试ReactNative的JS代码</b>

<li>Chrom 中打开 [http://localhost:8081/debugger-ui/](http://localhost:8081/debugger-ui/ ) 

<li>第一次运行的话需要安装React-devtools ```npm install -g react-devtools``` 要VPN翻墙的 <br>也可参照:[https://github.com/facebook/react-devtools/tree/master/packages/react-devtools](https://github.com/facebook/react-devtools/tree/master/packages/react-devtools)

<li>安装完成之后运行 ```react-devtools```

*这里有个问题是我一直遇到 Electorn failed to install correctly* 这个问题.

<li>Chrom 添加开发调式插件
https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi 

###2.2 将原生代码移植到ReactNative

###2.3 一些Android相关操作

<b>应用签名打包</b>

1. 生成签名文件
2. 配置Gradle中的relesase
3. 利用Gradle命令打包

<b>库版本升级</b>


##3 基础控件
##4 自定义控件
##5 项目架构
##6 项目实战


[逻辑性最强的React Native环境搭建与调试](http://www.cnblogs.com/vipstone/p/7112569.html)