#前言
最近项目中用得数据库框架 <b>“ActivieAndroi”</b> 由于作者停止维护了，它在升级到8.0之后会发生Crash，因此，我们准备给项目换一个数据库。主要考虑了Relam,Room,GreendDao 三个开源得数据库框架。

#内容

## Room
	Google 支持的ARCH框架推荐；使用原生SQL；注解
### 使用
网上一半教程都是在翻译官方文档，有兴趣的可以自己去看下官方给的Document & Demo

Git：[https://github.com/googlesamples/android-architecture-components/](https://github.com/googlesamples/android-architecture-components/)
Document：[https://developer.android.com/training/data-storage/room/](https://developer.android.com/training/data-storage/room/)

<b>1 引用Room库</b></br>
```
dependencies {
    def room_version = "1.1.0" // or, for latest rc, use "1.1.1-rc1"

    implementation "android.arch.persistence.room:runtime:$room_version"
    annotationProcessor "android.arch.persistence.room:compiler:$room_version"

    // optional - RxJava support for Room
    implementation "android.arch.persistence.room:rxjava2:$room_version"

    // optional - Guava support for Room, including Optional and ListenableFuture
    implementation "android.arch.persistence.room:guava:$room_version"

    // Test helpers
    testImplementation "android.arch.persistence.room:testing:$room_version"
}
```

<b>2 定义数据表实体类</b></br>
```
@Entity(tableName = "user_tb")
public class UserRoom {

    @PrimaryKey(autoGenerate = true)
    private int uid;
    @ColumnInfo
    private String name;
    @ColumnInfo
    private int age;
    @ColumnInfo
    private Date birthday;
 	... 省略了get/set
}
```

<b>3 编写Dao</b></br>
直接看Code吧，因为是ORM数据库，注解当然少不了。是不是有点想Retrofit得写法？而且支持写原生SQL，我觉得是很Nice得！别特么说Android程序员不需要学SQL，没点底子出了问题你都不晓得咋搞的。

```
@Dao
public interface UserDao {

    @Insert
    Long insertUser(UserRoom user);

    @Query("SELECT * FROM user_tb")
    Flowable<List<UserRoom>> queryAll();

    @Query("SELECT * FROM user_tb")
    List<UserRoom> queryForAll();

    @Query("SELECT * FROM user_tb WHERE name = :name")
    int deleteByName(String name);

    @Query("SELECT * FROM user_tb WHERE datetime(birthday/1000, 'unixepoch') >= datetime('now','-1 hour') ")
    Flowable<List<UserRoom>> queryWithinHour();

   	@Delete
	void deleteUser(UserRoom user);

	@Query("DELETE FORM user_tb where uid = :id")
	void deleteUserById(int id)
	
}
```

<b>4 在NApplication中初始化</b></br>
写一个类继承RoomDatabaes，我是写了一个单利.
```
// 要生成的数据表实例类
@Database(entities = {UserRoom.class},version = 1,exportSchema = false)
// 如果要存储Date数据，需要加这个来转换
@TypeConverters({DateConverts.class})
public abstract class AndroidDataBases extends RoomDatabase {

    public abstract UserDao userDao();

    private static volatile AndroidDataBases INSTANCE;

    public static AndroidDataBases crateDatabases(Context context) {
        if (INSTANCE == null) {
            synchronized (AndroidDataBases.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(),
                            AndroidDataBases.class, "room.db")
                            .build();
                }
            }
        }
        return INSTANCE;
    }

    public static AndroidDataBases getInstance(){
        return INSTANCE;
    }
}


public class DateConverts {
    @TypeConverter
    public static Date revertDate(Long value) {
        return value == null ? null : new Date(value);
    }

    @TypeConverter
    public static Long converterDate(Date date) {
        return date == null ? null : date.getTime();
    }
}
```

### CURD

有一点需要注意得是Room默认是必须在子线程操作得，否则抛异常。

```
	// 先获取一个UserDao
    var userDao: UserDao = AndroidDataBases.getInstance().userDao()().userDao();

	// 插入一条数据
	val user: UserRoom = UserRoom("admin2", 25, Date(System.currentTimeMillis()))
            AsyncTask.execute({
                var raw = userDao.insertUser(user)
                Log.d("picher","操作行号:"+raw)
            })
	// 查询数据
	AsyncTask.execute(Runnable {
                 var raws = userDao.queryForAll()
                 Log.d("picher","数据库："+raws.size)
             })
	// 删除数据 & 修改数据类似 只要在Dao中写好了直接调
	
```

看下用了Rx得写法(有坑，插入数据时也会触发这个回掉)：</br>

首先需要引用Room 自己的RX库。 
```
// RxJava support for Room
    compile "android.arch.persistence.room:rxjava2:1.1.1-rc1"
```

```
 private fun flowableDemo() {
           userDao.queryAll().subscribeOn(Schedulers.io())
                   .map({ users -> 
					   var names = ""
                       for (item in users) names += item.name
                       names
                   }).observeOn(AndroidSchedulers.mainThread()).subscribe({ names -> showTv.text =names})
    }

```

### 数据迁移
用Integer，如果用int得话，再迁移得时候会有一个问题如下。
```Migration didn't properly handle conversion```
然后仔细去对比异常下得信息，会发现，是刚添加得int 型字段导致得，我们添加得时候没有指定Not Null 但是系统似乎给我们加上了 Not Null = true 所以出错了。

### Room的坑
1. 实例要给一个空参构造.
2. 要支持存储Date类型的数据必须加@DateConvert.
3. 和Rx一起用的时候需要引用Room的Rx库.
4. Rx在查询完记得dispose，否则每次操作操作数据库都会收到回掉

暂时就遇到这些坑，之后如果遇到再来补上。大家慎踩...

#总结
个人感觉上手还是不错的，开发上也比较爽，还能练写原生SQL，有Retrtofit使用经验的上手更快，只是大项目先别急着上，毕竟才出来，先把坑排一排在用。