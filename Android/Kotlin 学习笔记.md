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

### 条件控制
val a = if (2>1) "对了" else "错了"
使用when 来代替switch case ，if else
```
when(4){
	in 1,0 - > ""
	else - > ""
}
```

### 循环
for in、while、do while
支持 break、continue
```
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (……) break@loop
    }
}
```
```
 val ints = arrayOf(1,2,0,3,5,6)
        ints.forEach lit@ {
            if (it == 0) return@lit
            Log.d("loopDemo", "$it")
        }
```
return@lit 类似continue

```
 sie@ for(i in 1..10){
            if(i == 5) break@sie
            Log.d("brak@", "$i")
        }
```
break@sie 类似return 不会执行之后的循环

### 类
```
class Student(str1:String,ageS:Int,heighIn:Int){
        var name:String = str1
            get() = field.toUpperCase()
            set

        var age:Int = ageS
            get() = field
            set(value) = if (value <= 100) {
                field = value
            } else {
                field = 0
            }
        var height:Int = heighIn // 私有化set
            private set
    }
    var aaa: Int = 0
            get() {
                field // 这里必须出现一下field关键字，否则 var aaa: Int = 0 会报错，除非你去掉 = 0这部分，不要给它赋初始化值
                return 0
            }
            set(value) {}
```
构造，成员变量
KT没有new，直接引用var s = Student()

**lateinit关键字**
延迟初始化，因为kt中变量需要再定意时初始化，对于一些再使用时初始化的变量，使用lateinit修饰

**次构造函数**
```
// 次构造函数示例
    class SClass(var name: String){
        lateinit var grade:String
        constructor(name: String,grade:String) : this(name){
            this.grade = grade
        }
    }
```
嵌套类，内部类，匿名内部类

**类修饰符**
abstract    // 抽象类  
final       // 类不可继承，默认属性
enum        // 枚举类
open        // 类可继承，类默认是final的
annotation  // 注解类

类的继承
如果子类没有主构造函数，则必须在每一个二级构造函数中用 super 关键字初始化基类，或者在代理另一个构造函数。初始化基类时，可以调用基类的不同构造方法。

**接口**
不同接口可以使用相同的方法名，用泛型区分；
```
interface A {
        fun foo() { print("a") }
    }

    interface B {
        fun foo() { print("b") }
    }

    class C:A,B{
        override fun foo() {
            super<A>.foo()
            super<B>.foo()
        }
    }
```

###Kotlin 扩展
方法扩展，熟悉扩展
```
 val Student.fee:String
        get() = "扩展属性"

    fun Student.judgeFee(){
        print("扩展：计算学生费用$fee")
    }
```
扩展属性不能被初始化

### Kotlin 泛型
泛型类，泛型方法
```
	/**
     * 泛型类,泛型方法
     */
    class Box<T>(t:T){
        var value = t

        fun <T> values(): String {
            return "$value 111"
        }
    }

    val Student.fee:String
        get() = "扩展属性"

    fun Student.judgeFee(){
        println("扩展：计算学生${name}费用$fee")
    }
```
泛型约束，星号投射

### 匿名内部类、对象表达式、对象声名、伴生对象
对象表达式：使用得时候初始化
```
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }
    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

对象声名：第一次访问到，延迟初始化
```
 // 对象声明
    object Site{
        var url = "www.baidu.com"
        var naem = "百度"
    }
	...
  		val s1 = Site
        Log.d("picher","s1 toString：$s1")
        Log.d("picher","name:${s1.naem},url:${s1.url}")
        val s2 = Site
        s2.url = "www.yahoo.com"
        Log.d("picher","name:${s2.naem},url:${s2.url}")
```

伴生对象：类被加载时被初始化
```
 	/**
     * 线程安全的单利
     */
    class Exame private constructor(){
        companion object {
            private var instance: Exame? = null
                get()  {
                    if (field==null){
                        instance = Exame()
                    }
                    return field
                }
            @Synchronized
            fun get():Exame{
                return instance!!
            }
        }
    }
```
### 委托
类委托
```
	interface EntrustBase{
        fun print()
    }

    class EntrustBaseImpl(val b:Int): EntrustBase {
        override fun print() {
            print("Entrust：$b")
        }
    }
    // 类委托
    class Driver(b:EntrustBase):EntrustBase by b
```

属性委托
```
	// 属性委托 可以监控属性状态
    class DelegateAttribute{
        var p: String by Delegate()
    }

    class Delegate{
        operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
            return "$thisRef, 这里委托了 ${property.name} 属性"
        }

        operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
            Log.d("deleteDemo","$thisRef 的 ${property.name} 属性赋值为 $value")
        }
    }
```

lazy的使用，只赋值执行一次，之后调用只返回记录的结果
*注意一点，使用lazy之后 变量需要用val修饰，因为lazy没有实现setValue()的方法*

```
	val b:String by lazy {
            println("computed!")     // 第一次调用输出，第二次调用不执行
            "Hello"
        }
```
观察者，观测属性变化
```
private fun observableDemo() {
        val u = User()
        u.name = "张三"
        u.name = "李四"
    }

    // 监听属性改变
    class User{
        var name:String by Delegates.observable("初始值"){
            property, oldValue, newValue ->
            Log.d("observableDemo","property: $property oldValue:$oldValue newValue:$newValue")
        }
    }
```

属性映射
```
  private fun mappingDemo() {
        var m1 = MappingBen(mutableMapOf(
                "name" to "张三",
                "age" to 111
        ))
        Log.d("mappingDemo","name:${m1.name},age:${m1.age}")
    }

    // 代理属性不能使用get() set()
    class MappingBen(map:MutableMap<String,Any?>){
        var name:String by map
        var age:String by map
    }
```

提供委托



## Kotlin与Java 一些写法上的比较 
1 长字符串KT引入了三引号
2 方法参数默认值，减少函数重载
3 拓展函数，自定义类方法；拓展属性
4 委托，简洁的实现装饰者模式
5 with，减少显示调用 apply，类名直接调用，域内可调用类的公有方法
6 ?.let{} Null 校验, ?: 左边null 则执行右边 类似三元
7 操作符重载 

# 总结
毕竟是一门新语言，和JAVA还是很不一样的，省略了很多不必要得代码，但是新手开始的时候可能还是会感觉不太好阅读。这次只记录了一些基本的使用方法，具体到项目中使用的时候才能更有感受！

