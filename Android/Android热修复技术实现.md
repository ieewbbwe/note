#热修复学习思维笔记—基于Tinker


## 概述
热修复去年就开始火了，主要解决的是APP在更新时需要重新下载整个APK的不友好用户体验。比如，线上的应用出现了一个严重bug，开发人员修复之后还要上传到市场用户重新下载安装，大多数用户都不太愿意再去重新下载，即使是微信，支付宝这种应用开发出新功能想要推广，也不得不花大力气让用户更新版本。

针对这个问题有两个解决方案：

1、WebApp 用H5混合开发，也是目前一些应用用的最多的。

优点：更新速度快  
缺点：耗费流量，网速慢则要加载很久，体验不好

2、 热修复 利用安卓dex分包的特性，使用增量包更新应用。

优点：原生应用体验良好  
缺点：只适用于少量逻辑的修复

H5暂且不谈，今天基于微信团队开源的tinker记录一下学习时的思维路线。

<B>首先搞明白什么是热修复，和他的实现原理</B>

最开始接触这个还是一脸懵逼，只知道能干啥，不明白具体咋做的。那就各种找资料。

（初始文章一定要看，不然都搞不明白他是咋做的，余下两篇我会简要概括实现步奏，可以先不必看）

安卓App热补丁动态修复技术介绍 初始文章
https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a&scene=1&srcid=1106Imu9ZgwybID13e7y2nEi#wechat_redirect

借鉴文章
http://www.tuicool.com/articles/367RBvY

实现文章
http://blog.csdn.net/lmj623565791/article/details/54882693

看完还懵逼的我简要概括一下：

<b>热修复思路，1 生成一个补丁包DEX 2 将补丁包与原有的apk合并，覆盖掉修改的方法</b>

具体还涉及的elements 中list的选择，CLASS_ISPREVERIFILE标记的解决，想深入了解在去看。

## 使用

了解了大致的原理我们就来说一下实现步奏，笔者是基于Tinker来实现热修复的，至于为啥选择这个，大家用过都说好，我也要用。

有两种实现方式：1 基于命令行 2基于Gradle配置

其实思路都一样，只是操作步奏不同，我就主要介绍基于Gradle的来说，因为目前基于Gradle的配置越来越多，要学会用它。

### 1 生成原包

这部分里因为涉及到一些配置可能步奏比较多。

<b>第一步 Project的Gradle中添加</b>

```
dependencies {
            classpath ('com.tencent.tinker:tinker-patch-gradle-plugin:1.7.5')
        }
```

<b>第二步 Module的Gradle中添加</b>

```
dependencies {
    //optional, help to generate the final application
    provided('com.tencent.tinker:tinker-android-anno:1.7.5')
    //tinker's main Android lib
    compile('com.tencent.tinker:tinker-android-lib:1.7.5')
    }
```

别加错地方了啊，Project包含Module，加错编译不了。

同时添加一些Tinker 基础配置，比较多，直接复制去用，不用一行一行看。

```

/**Tinker 相关配置----------------开始-----------------------------------*/
def bakPath = file("${buildDir}/bakApk/")

def gitSha() {
    try {
        String gitRev = '11.2.3.4'
        if (gitRev == null) {
            throw new GradleException("can't get git rev, you should add git to system path or just input test value, such as 'testTinkerId'")
        }
        return gitRev
    } catch (Exception e) {
        throw new GradleException("can't get git rev, you should add git to system path or just input test value, such as 'testTinkerId'")
    }
}

ext {
    //for some reason, you may want to ignore tinkerBuild, such as instant run debug build?
    //Tinker是否可用
    tinkerEnabled = true
    //for normal build
    //old apk file to build patch apk 旧包名
    tinkerOldApkPath = "${bakPath}/app-debug-0222-15-25-40.apk"
    //proguard mapping file to build patch apk  旧包混淆文件目录
    tinkerApplyMappingPath = "${bakPath}/app-debug-1018-17-32-47-mapping.txt"
    //resource R.txt to build patch apk, must input if there is resource changed 旧包R文件
    tinkerApplyResourcePath = "${bakPath}/app-debug-0222-15-25-40-R.txt"
    //only use for build all flavor, if not, just ignore this field  多渠道
    tinkerBuildFlavorDirectory = "${bakPath}/app-1018-17-32-47"
}

def getOldApkPath() {
    return hasProperty("OLD_APK") ? OLD_APK : ext.tinkerOldApkPath
}

def getApplyMappingPath() {
    return hasProperty("APPLY_MAPPING") ? APPLY_MAPPING : ext.tinkerApplyMappingPath
}

def getApplyResourceMappingPath() {
    return hasProperty("APPLY_RESOURCE") ? APPLY_RESOURCE : ext.tinkerApplyResourcePath
}

def getTinkerIdValue() {
    return hasProperty("TINKER_ID") ? TINKER_ID : gitSha()
}

def buildWithTinker() {
    return hasProperty("TINKER_ENABLE") ? TINKER_ENABLE : ext.tinkerEnabled
}

def getTinkerBuildFlavorDirectory() {
    return ext.tinkerBuildFlavorDirectory
}

if (buildWithTinker()) {
    apply plugin: 'com.tencent.tinker.patch'
    tinkerPatch {
        /**
         * 基准apk包路径，也就是旧包路径
         * */
        oldApk = getOldApkPath()
        /**
         * 如果出现以下的情况，并且ignoreWarning为false，我们将中断编译。因为这些情况可能会导致编译出来的patch包
         * 带来风险：
         * 1. minSdkVersion小于14，但是dexMode的值为"raw";
         * 2. 新编译的安装包出现新增的四大组件(Activity, BroadcastReceiver...)；
         * 3. 定义在dex.loader用于加载补丁的类不在main dex中;
         * 4. 定义在dex.loader用于加载补丁的类出现修改；
         * 5. resources.arsc改变，但没有使用applyResourceMapping编译。
         * */
        ignoreWarning = false
        /**
         * 在运行过程中，我们需要验证基准apk包与补丁包的签名是否一致，我们是否需要为你签名
         * */
        useSign = true

        buildConfig {
            /**
             * 可选参数；在编译新的apk时候，我们希望通过保持旧apk的proguard混淆方式，从而减少补丁包的大小。这个只
             * 是推荐的，但设置applyMapping会影响任何的assemble编译。
             * */
            applyMapping = getApplyMappingPath()
            /**
             * 可选参数；在编译新的apk时候，我们希望通过旧apk的R.txt文件保持ResId的分配，这样不仅可以减少补丁包的
             * 大小，同时也避免由于ResId改变导致remote view异常。
             * */
            applyResourceMapping = getApplyResourceMappingPath()
            /**
             * 在运行过程中，我们需要验证基准apk包的tinkerId是否等于补丁包的tinkerId。这个是决定补丁包能运行在哪些
             * 基准包上面，一般来说我们可以使用git版本号、versionName等等。
             * */
            tinkerId = getTinkerIdValue()
        }
        dex {
            /**
             * 只能是'raw'或者'jar'。
             * 对于'raw'模式，我们将会保持输入dex的格式。
             * 对于'jar'模式，我们将会把输入dex重新压缩封装到jar。如果你的minSdkVersion小于14，你必须选择‘jar’模式
             * ，而且它更省存储空间，但是验证md5时比'raw'模式耗时()
             * */
            dexMode = "jar"
            /**
             * 是否提前生成dex，而非合成的方式。这套方案即回退成Qzone的方案，对于需要使用加固或者多flavor打包(建
             * 议使用其他方式生成渠道包)的用户可使用。但是这套方案需要插桩，会造成Dalvik下性能损耗以及Art补丁包可
             * 能过大的问题，务必谨慎使用。另外一方面，这种方案在Android N之后可能会产生问题，建议过滤N之后的用户。
             */
            usePreGeneratedPatchDex = false
            /**
             * 需要处理dex路径，支持*、?通配符，必须使用'/'分割。路径是相对安装包的，例如/assets/...
             */
            pattern = ["classes*.dex",
                       "assets/secondary-dex-?.jar"]
            /**
             *     这一项非常重要，它定义了哪些类在加载补丁包的时候会用到。这些类是通过Tinker无法修改的类，也是一定要放在main dex的类。
             这里需要定义的类有：
             1. 你自己定义的Application类；
             2. Tinker库中用于加载补丁包的部分类，即com.tencent.tinker.loader.*；
             3. 如果你自定义了TinkerLoader，需要将它以及它引用的所有类也加入loader中；
             4. 其他一些你不希望被更改的类，例如Sample中的BaseBuildInfo类。这里需要注意的是，这些类的直接引用类也
             需要加入到loader中。或者你需要将这个类变成非preverify。
             */
            loader = ["com.tencent.tinker.loader.*",
                      //warning, you must change it with your application
                      //TODO 换成自己的Application
                      "com.webber.tinkerdemo.SampleApplication",
            ]
        }
        lib {
            /**
             * 需要处理lib路径，支持*、?通配符，必须使用'/'分割。与dex.pattern一致, 路径是相对安装包的，例如/assets/...
             */
            pattern = ["lib/armeabi/*.so"]
        }
        res {
            /**
             * 需要处理res路径，支持*、?通配符，必须使用'/'分割。与dex.pattern一致, 路径是相对安装包的，例如/assets/...，
             * 务必注意的是，只有满足pattern的资源才会放到合成后的资源包。
             */
            pattern = ["res/*", "assets/*", "resources.arsc", "AndroidManifest.xml"]
            /**
             * 支持*、?通配符，必须使用'/'分割。若满足ignoreChange的pattern，在编译时会忽略该文件的新增、删除与修改。
             * 最极端的情况，ignoreChange与上面的pattern一致，即会完全忽略所有资源的修改。
             */
            ignoreChange = ["assets/sample_meta.txt"]
            /**
             * 对于修改的资源，如果大于largeModSize，我们将使用bsdiff算法。这可以降低补丁包的大小，但是会增加合成
             * 时的复杂度。默认大小为100kb
             */
            largeModSize = 100
        }
        packageConfig {
            /**
             * configField("key", "value"), 默认我们自动从基准安装包与新安装包的Manifest中读取tinkerId,并自动
             * 写入configField。在这里，你可以定义其他的信息，在运行时可以通过TinkerLoadResult.getPackageConfigByName得到相应的数值。但是建议直接通过修改代码来实现，例如BuildConfig。
             */
            configField("patchMessage", "tinker is sample to use")
        }
        sevenZip {
            /**
             * 例如"com.tencent.mm:SevenZip:1.1.10"，将自动根据机器属性获得对应的7za运行文件，推荐使用
             */
            zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
        }
        /**
         *  文件名                              描述
         *  patch_unsigned.apk                  没有签名的补丁包
         *  patch_signed.apk                  签名后的补丁包
         *  patch_signed_7zip.apk              签名后并使用7zip压缩的补丁包，也是我们通常使用的补丁包。但正式发布的时候，最好不要以.apk结尾，防止被运营商挟持。
         *  log.txt                              在编译补丁包过程的控制台日志
         *  dex_log.txt                          在编译补丁包过程关于dex的日志
         *  so_log.txt                          在编译补丁包过程关于lib的日志
         *  tinker_result                      最终在补丁包的内容，包括diff的dex、lib以及assets下面的meta文件
         *  resources_out.zip                  最终在手机上合成的全量资源apk，你可以在这里查看是否有文件遗漏
         *  resources_out_7z.zip              根据7zip最终在手机上合成的全量资源apk
         *  tempPatchedDexes                  在Dalvik与Art平台，最终在手机上合成的完整Dex，我们可以在这里查看dex合成的产物。
         *
         *
         * */

        /**
         * 获得所有渠道集合，并判断数量
         */
        List<String> flavors = new ArrayList<>();
        project.android.productFlavors.each { flavor ->
            flavors.add(flavor.name)
        }
        boolean hasFlavors = flavors.size() > 0
        /**
         * bak apk and mapping
         *  创建Task并执行文件操作
         */
        android.applicationVariants.all { variant ->
            /**
             * task type, you want to bak
             */
            def taskName = variant.name
            def date = new Date().format("MMdd-HH-mm-ss")

            tasks.all {
                if ("assemble${taskName.capitalize()}".equalsIgnoreCase(it.name)) {

                    it.doLast {
                        copy {
                            def fileNamePrefix = "${project.name}-${variant.baseName}"
                            def newFileNamePrefix = hasFlavors ? "${fileNamePrefix}" : "${fileNamePrefix}-${date}"

                            def destPath = hasFlavors ? file("${bakPath}/${project.name}-${date}/${variant.flavorName}") : bakPath
                            from variant.outputs.outputFile
                            into destPath
                            rename { String fileName ->
                                fileName.replace("${fileNamePrefix}.apk", "${newFileNamePrefix}.apk")
                            }

                            from "${buildDir}/outputs/mapping/${variant.dirName}/mapping.txt"
                            into destPath
                            rename { String fileName ->
                                fileName.replace("mapping.txt", "${newFileNamePrefix}-mapping.txt")
                            }

                            from "${buildDir}/intermediates/symbols/${variant.dirName}/R.txt"
                            into destPath
                            rename { String fileName ->
                                fileName.replace("R.txt", "${newFileNamePrefix}-R.txt")
                            }
                        }
                    }
                }
            }
        }
        /**
         * 如果有渠道则进行多渠道打包
         */
        project.afterEvaluate {
            //sample use for build all flavor for one time
            if (hasFlavors) {
                task(tinkerPatchAllFlavorRelease) {
                    group = 'tinker'
                    def originOldPath = getTinkerBuildFlavorDirectory()
                    for (String flavor : flavors) {
                        def tinkerTask = tasks.getByName("tinkerPatch${flavor.capitalize()}Release")
                        dependsOn tinkerTask
                        def preAssembleTask = tasks.getByName("process${flavor.capitalize()}ReleaseManifest")
                        preAssembleTask.doFirst {
                            String flavorName = preAssembleTask.name.substring(7, 8).toLowerCase() + preAssembleTask.name.substring(8, preAssembleTask.name.length() - 15)
                            project.tinkerPatch.oldApk = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-release.apk"
                            project.tinkerPatch.buildConfig.applyMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-release-mapping.txt"
                            project.tinkerPatch.buildConfig.applyResourceMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-release-R.txt"

                        }

                    }
                }

                task(tinkerPatchAllFlavorDebug) {
                    group = 'tinker'
                    def originOldPath = getTinkerBuildFlavorDirectory()
                    for (String flavor : flavors) {
                        def tinkerTask = tasks.getByName("tinkerPatch${flavor.capitalize()}Debug")
                        dependsOn tinkerTask
                        def preAssembleTask = tasks.getByName("process${flavor.capitalize()}DebugManifest")
                        preAssembleTask.doFirst {
                            String flavorName = preAssembleTask.name.substring(7, 8).toLowerCase() + preAssembleTask.name.substring(8, preAssembleTask.name.length() - 13)
                            project.tinkerPatch.oldApk = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-debug.apk"
                            project.tinkerPatch.buildConfig.applyMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-debug-mapping.txt"
                            project.tinkerPatch.buildConfig.applyResourceMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-debug-R.txt"
                        }

                    }
                }
            }
        }
    }
}
/**Tinker 相关配置----------------结束-----------------------------------*/
```

有中文注释，我就不介绍了，注意tinkerId这块容易抛出一个tinkerId is not set！！的错误。

这个tinkerId 是Tinker用来判断要更新哪个包的标记，一般用版本号或者git版本信息来生成，我这里写死的，添加git配置之后，将 String gitRev = '11.2.3.4' 换成        String gitRev = 'git rev-parse --short HEAD'.execute(null, project.rootDir).text.trim()。  意思就是去拿仓库的版本信息

<b>第三步 创建一个SampleApplication类，继承DefaultApplicationLike</b>

```
@DefaultLifeCycle(application = "com.webber.tinkerdemo.SampleApplication",
        flags = ShareConstants.TINKER_ENABLE_ALL,
        loadVerifyFlag = false)
public class SampleApplicationLike extends DefaultApplicationLike {
    public SampleApplicationLike(Application application, int tinkerFlags, boolean tinkerLoadVerifyFlag, long applicationStartElapsedTime, long applicationStartMillisTime, Intent tinkerResultIntent, Resources[] resources, ClassLoader[] classLoader, AssetManager[] assetManager) {
        super(application, tinkerFlags, tinkerLoadVerifyFlag, applicationStartElapsedTime, applicationStartMillisTime, tinkerResultIntent, resources, classLoader, assetManager);
    }

    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    @Override
    public void onBaseContextAttached(Context base) {
        super.onBaseContextAttached(base);
        TinkerInstaller.install(this);
    }

    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    public void registerActivityLifecycleCallbacks(Application.ActivityLifecycleCallbacks callback) {
        getApplication().registerActivityLifecycleCallbacks(callback);
    }
}
```

注意```application = "com.webber.tinkerdemo.SampleApplication"```这里要换成你配在manifest里面的那个地址，这里建议你们就把包名改一下就可以了，这个类本身Tinker会帮你去创建。

同时在manifest里面配置

```<application android:name=".SampleApplication">```

注意下刚刚复制的大段配置里面有一个//TODO那里，要改为自己Application的地址 本例中也就是```com.webber.tinkerdemo.SampleApplication```

<b>第四步 点击Build->Build APK 生成原包</b>

![这里写图片描述](http://img.blog.csdn.net/20170222182554979?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW94aW91aGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


打包后会生成如下目录：

![这里写图片描述](http://img.blog.csdn.net/20170222182713872?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW94aW91aGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在这里将包拷贝到手机安装（当然你也可以直接Generate Signed Apk） 

### 2 生成修改后的包

现在我们可以随意改动一点代码，比如我改一下按钮的名称。

之后配置一下Gradle中tinkerOldApkPath和tinkerApplyResourcePath（如果有混淆文件的话tinkerApplyMappingPath也要配置），配置为你原包的名字即可。

![这里写图片描述](http://img.blog.csdn.net/20170222182806794?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW94aW91aGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 3 比较新旧包生成补丁包

然后点击Studio面板最右侧的Gradle,如图，如果你刚才dependencies都添加正确的话，这里会有如图的目录。

![这里写图片描述](http://img.blog.csdn.net/20170222182616542?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW94aW91aGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们点击一下tinkerPathDeBug，他就会自动去帮我们比较包中的新增项，并在```app->build->outputs```中新增一个tinkerPatch的文件夹，里面放的是一些比较操作产生的文件，我们将patch_signed_7zip.apk拷贝到设备跟目录

### 4 修复测试

准备OK我们来测试一下结果吧。

![这里写图片描述](http://img.blog.csdn.net/20170222182102055?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW94aW91aGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

哈哈其实点一下Load 就可以了，下面几个按钮还没有写方法，Load按钮对应的方法```        TinkerInstaller.onReceiveUpgradePatch(getApplicationContext(), Environment.getExternalStorageDirectory().getAbsolutePath() + "/patch_signed_7zip.apk");```

OK完成，我们再来看一下原包和补丁包的大小

![这里写图片描述](http://img.blog.csdn.net/20170222182300100?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW94aW91aGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

哈哈是不是很爽，补丁包很小，自己应用里面比如放在初始化的时候请求一下看是否有补丁，有就提示用户下载更新，超级方便，再也不用再去重复下载啦~哈哈 坑了运营妹子的下载量请吃一顿饭就OK了~

至于第一种实现方法也很简单，去看我上文提供的blog吧，我就不演示了，要学会使用Gradle哈哈。
 


