#Could not resolve com.android.supportsupport-annotations25.4.0
#概述
最近在做客户项目的维护工作，才进项目，那边的同事不知道要干啥把Support升级到了25.4.0，我给看懵逼了。
Sdk manager 都找不到的。
不信可以试试

``` androidTestCompile "com.android.support:support-annotations:25.4.0"```

#解决办法
查了无数资料，有以下几种答案

<b>1 降低版本吧，你这个太高了</b>

确实一般情况下降一个现行的版本就ok了，但是这个项目不晓得客户用的啥东西，降低版本后还有别的一堆错误。

<b>2 Sdk Manager里面更新一下Libraried Repository</b>

然而，目前最新的就是47，也没有25.4.0，可以到  

..\extras\android\m2repository\com\android\support\support-annotations

这个目录下自己去看，也是没有的

去看官网文档？拿好不谢，然后他也是让你更新库，降低最高版本。没啥卵用

[https://developer.android.com/topic/libraries/support-library/revisions.html#25-4-0  ](https://developer.android.com/topic/libraries/support-library/revisions.html#25-4-0  )

<b>3 依赖Google的 Maven 库</b>

搞得时间长了也找到了答案，这个库是google放在自己的maven维护的，需要依赖他的库。在根目录build.gradle中写入如下代码

```
allprojects {
    repositories {
        jcenter()
        maven {
            url 'https://maven.google.com/'
            name 'Google'
        }
        mavenCentral()
    }
}

```

OK！搞定，继续开心的codding ~~