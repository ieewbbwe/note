# 前言
要保持一颗学习的心，才能厚积薄发。Kotlin已经出来有一段时间了，市面上也有一些Android开发开始吃螃蟹，褒贬不一。官网标榜的KT能让代码更加简洁，开发效率更高，吸引力也很大。抱着充实自己的态度，姑且学习一下！

*这篇文档再18年3月的时候就创建了，中途各种事情，自己也有些拖拉一直没有继续写，趁着现在有点时间，把笔记完善一下，也可以给自己一个参考。*

# 内容
将从如何搭建环境，语法糖，最后用一个登陆、列表页面的简单例子来总结。

## Kotlin 环境配置
Android Studio 从 3.0（preview）版本开始将内置安装 Kotlin 插件。
1. 安装Kotlin插件
2. 再module的build.gradle 里面加上
```apply plugin: 'kotlin-android'``` 
3. 再项目的build.gradle 里面加上 
```classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.2.41"```

## Koltin 语法糖
### 变量
var 可变变量，val 不可变变量定义

### 字符串模板
可以再**""** 包裹的字符串里面使用**$**来引用变量

```
private fun stringComp(){
        var a = 1
        val s1 = "a is $a"
        a = 2
        val s2 = "${s1.replace("is","was")},but not $a"
        print("stringComp:$s2")
    }

```
### Null 检验机制
!! is Null then throw NullExeception
? is Null then return null

### 区间
```
 for (i in 1..4) {
            Log.d("picher", "1..4 $i")
        }

        for (i in 1..4 step 2) {
            Log.d("picher", "1..4 step 2,$i")
        }

        for (i in 4 downTo 1) {
            Log.d("picher", "4 down to ,$i")
        }
```

### 基础数据类型
**===** 比较对象地址，**==** 比较数值 

## Kotlin与Java 一些写法上的比较 
1 长字符串KT引入了三引号
2 方法参数默认值，减少函数重载
3 拓展函数，自定义类方法；拓展属性
4 委托，简洁的实现装饰者模式
5 with，减少显示调用 apply，类名直接调用，域内可调用类的公有方法
6 ?.let{} Null 校验, ?: 左边null 则执行右边 类似三元
7 操作符重载 


# 总结

