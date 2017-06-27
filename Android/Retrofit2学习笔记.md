#Retrofit 笔记

## 概述

因为在Android 5.1中，org.apache.http包中的类和AndroidHttpClient类均已被废弃，至于为什么？http://blog.csdn.net/guolin_blog/article/details/12452307
去看吧，郭大神的写的，但不是本文重点。

随后google推荐的OkHttp，和在之后基于Okhttp出现的Retrofit开始被大家使用。这篇文章就来记录一下Retrofit的一些用法。以后自己找起来也方便。

##怎么用

### Get请求

http://localhost/user
@GET("user")

### Post请求
