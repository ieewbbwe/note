#前言
接上一张Room，Leader说Room作为一个新框架才出来不久，坑应该不少！（无法反驳- -~！！）所以我们采用Realm，并且这玩意三端都可以用（神特么，以为写一套代码适配三端？？），总之一堆理由，So，抱着对学习无比的热爱和对知识的渴求，（受不了Leader BB被逼换库- -！）来填一填Realm的坑。

#内容
## Realm
	开源；效率高；跨平台，Android、IOS、前端、后台、PC都可以用，但是代码会有所不同
	Git:https://github.com/realm/realm-cocoa
### 使用
网上有很多教程，大多是翻译官网教程的，想详细了解可以自行Google，这里只说通常大家的用法。

因为他是作为一个插件被引用的，所以首先添加</br>

<b>1 在Project build.gradle 添加ClassPath</b></br>

```
 dependencies {
		...
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath "io.realm:realm-gradle-plugin:5.1.0"
		...
	}
```

<b>2 在Module build.gradle 添加 plugin</b>

```
apply plugin: 'realm-android'
```

<b>3 初始化数据库</b></br>
写一个NApplication 继承Application，然后替换manifest里面name为自己的Application。
```
 <application
        android:name=".NApplication"
		...
	/>
```
在NApplicaiotn onCreate()里面初始化数据库。

```
		Realm.init(this);
        RealmConfiguration config = new RealmConfiguration.Builder()
                .name("realm_demo.realm")
                .rxFactory(new RealmObservableFactory())
                .schemaVersion(1)
                .build();
        Realm.setDefaultConfiguration(config);
        try {
            Realm.migrateRealm(config, new Migration());
        } catch (FileNotFoundException ignored) {
            // If the Realm file doesn't exist, just ignore.
        }
```


OK，编译一下就可以用了。

### CURD

<b>1 首先看下声明对象。有两种选择</b>

1. extends RealmObject
2. implements RealmModel 并且添加 @RealmClass

```
@RealmClass
public class UserRealm implements RealmModel {
	...
}

// 或者

public class UserRealm extends RealmObject {
	...
}

```

不支持主键自增，需要添加主键的话要加```@PrimaryKey```

<b>2 增删改查的写法</b>

有很多种写法，这里只说通常用的写法，有executeTransaction和executeTransactionAsync。支持事务的一个异步一个不是异步。

```
 // 添加数据
 mRealm.executeTransaction({ realm ->
            val user = UserRealm("Webber", 25, Date(System.currentTimeMillis()))
            realm.insert(user)
        })

//查询数据
 mRealm.executeTransaction({ realm ->
            val users = realm.where(UserRealm::class.java).findAll()
        })
// 删除数据 & 更新数据 (查出来之后直接操作对象即可)

```
### 数据库升级及数据迁移
占坑


### Realm的坑
1. ID 不能自增
2. 实体类<b>只能</b> 继承RealmObjet 或者实现RealmModel加@RealmClass注解。
3. 查询出来的数据需要copyFromRealm()
4. Realm 的创建和关闭<b>必须</b>在同一个线程
5. 只有Query可以与RX连用，返回Flowable

参考:[Realm数据库的那些坑](https://blog.csdn.net/guoxiaolongonly/article/details/74295823)


#总结
参考：[【Android】Realm详解](https://www.jianshu.com/p/37af717761cc)
