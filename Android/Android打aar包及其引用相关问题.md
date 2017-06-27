
## 概述

最近手上有个项目是开发整个APP的部分功能，并以库的方式提供给主工程调用。开发中遇到了aar引用的一些问题，分享给大家，欢迎讨论。

## AAR打包

1 Build —> Clean Project -> ReBuild Project

2 在module的 build -> outputs -> aar 中就可以看到aar包了

注：前提是module 必须是 apply plugin: 'com.android.library'

aar和jar不同。

aar：包含所有资源，class以及res资源文件全部包含

jar：只包含了class文件与清单文件，不包含资源文件，如图片等所有res中的文件。 

## AAR引用

1 把aar包复制到module的libs里面

2 在module 中添加 

```
 repositories {
        flatDir {
            dirs 'libs'
        }
    }
```

3 添加依赖

```
    compile(name: 'you_aar_name', ext: 'aar')
```
 
4 Sync Project

一般情况下而言，这个aar包就可以正常工作了，你可以调用它里面的类了。

但是！！！万万没想到！！

aar这个货不包含依赖的资源！比如说我在lib包中  compile 'com.shuyu:GSYVideoPlayer:1.6.5'，但这个库里面的资源文件并不会被打到aar包中

在你编译的时候是没问题的，但是在运行的时候会出现ClassNotFoundException 的异常

http://stackoverflow.com/questions/29857141/using-aar-noclassdeffounderror-but-class-exists-and-is-dexed 

针对这个问题，有两个解决方案

<b>1. 将lib中依赖的库在module中重新依赖一下</b>
 
<b>2. 将lib放入JCenter或者maven中，让module引用</b> 

当然最简单的方法就是直接把lib库源码丢过去，module直接依赖，但是有些情况下你是不想让人看到或者更改lib中的代码的。比如当你提供给第三方的时候。

放在JCenter也可以，但是你的项目源码就会暴露了，对于企业级应用来说这样做很危险。

所以接下来我们说一种mavenLocal的方式来解决这个问题。

## 创建本地maven

Maven能够通过一小段描述信息来管理项目的构建。

它主要的就是pom文件，里面申明了仓库地址和依赖关系，我们这里要做的就是在编译的时候让编译器去我们指定的路径去搜索资源。

1 需要告诉Gradle本地maven的地址，在全局的Gradle中配置

```
allprojects {
    repositories {
        jcenter()
        mavenCentral()
        maven { url "../alib_mock_sdk/src/main/java" }
    }
}
```

2 创建出alib_mock_sdk

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.xxxxx.mobile.libs</groupId>
  <artifactId>alib</artifactId>
  <version>1.0.0</version>
  <packaging>aar</packaging>
    <dependencies>
    <dependency>
      <groupId>com.jakewharton</groupId>
      <artifactId>butterknife</artifactId>
      <version>7.0.1</version>
    </dependency>
    <dependency>
      <groupId>com.android.support</groupId>
      <artifactId>recyclerview-v7</artifactId>
      <version>23.4.0</version>
    </dependency>
    <dependency>
      <groupId>com.android.support</groupId>
      <artifactId>design</artifactId>
      <version>22.2.1</version>
    </dependency>
    <dependency>
      <groupId>com.android.support</groupId>
      <artifactId>support-v4</artifactId>
      <version>24.2.0</version>
    </dependency>
    <dependency>
      <groupId>com.shuyu</groupId>
      <artifactId>GSYVideoPlayer</artifactId>
      <version>1.6.5</version>
    </dependency>
    <dependency>
      <groupId>cn.woblog.android</groupId>
      <artifactId>downloader</artifactId>
      <version>1.0.1</version>
    </dependency>
  </dependencies>
</project>

```

3 在这个库的gradle中依赖aar

首先需要配置这个库的gradle,然后才能在dependencies依赖的时候找到这个地址

```
buildscript {
    repositories {
        mavenCentral()
        maven { url "./src/main/java" }
    }
}

repositories {
    mavenCentral()
    maven { url "./src/main/java" }
}


allprojects {
    repositories {
        mavenCentral()
        maven { url "./src/main/java" }
    }
}
```

```
compile 'com.xxxxx.mobile.libs:alib:1.0.0'
```

com.xxxxx.mobile.libs 对应刚pom中groupId

alib对应pom中artifactId

1.0.0对应pom中version

注意：这里注意Studio的一个坑，如果你是Compact Empty Middle Package 的时候邮件new 一个package Studio会创建出如下的目录，看似没有问题。但是你勾选上Flatten Packages的时候就会是如下的目录结构。

比如我这里想创建一个1.0.0 版本的包，但是可就会创建成1 1.0 1.0.0 三个，就导致你在依赖的时候路径不正确，死活找不到依赖文件！

4 在module的gralde中依赖这个库

```
compile project(':alib_mock_sdk')
```

到此就大功告成了，你可以在自己的项目中引用这个库文件了