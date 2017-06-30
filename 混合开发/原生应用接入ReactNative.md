#概述
最近在看ReactNative，这玩意一份代码可以编译成IOS、Android、WEB前端 三个平台上运行的程序。挺方便，但是项目不可能一下子全转为RN开发，于是就需要原生里面嵌入RN代码。

##一安装环境
安装ReactNative运行需要的环境。

### 1安装Python2
[https://www.python.org/](https://www.python.org/)
### 2安装Node
[http://nodejs.cn/](http://nodejs.cn/)

安装完node后建议设置npm镜像以加速后面的过程（或使用科学上网工具）。注意：不要使用cnpm！cnpm安装的模块路径比较奇怪，packager不能正常识别！在命令行依次运行如下命令：
```
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```
### 3安装react-native-cli

运行如下命令：
```npm install -g yarn react-native-cli```

安装完yarn后同理也要设置镜像源：
```
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```
### 4配置基础的Android开发环境
因为我是做Android开发的，环境已经搭建好了，主要就是要有Android Studio 并能跑起来程序。不会的去看吧[http://reactnative.cn/docs/0.45/getting-started.html#content](http://reactnative.cn/docs/0.45/getting-started.html#content)

注：然后你可以先用WebStorm写一个React Native项目感受一下。在准备接入原生。不会写的建议先去看一下[React](http://www.runoob.com/react/react-tutorial.html)，然后再看[ReactNative](http://reactnative.cn/docs/0.45/props.html#content)

##二系统配置
###1. 在module/build.gradle下添加依赖
```
compile 'com.facebook.react:react-native:0.45.1'
```
###2. 添加网络权限
```
<uses-permission android:name="android.permission.INTERNET" />
```
###3. 集成RN代码
```
public class MyReactActivity extends Activity implements DefaultHardwareBackBtnHandler {
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModuleName("index.android")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "MyAwesomeApp", null);

        setContentView(mReactRootView);
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }
}

@Override
protected void onPause() {
    super.onPause();

    if (mReactInstanceManager != null) {
        mReactInstanceManager.onPause();
    }
}

@Override
protected void onResume() {
    super.onResume();

    if (mReactInstanceManager != null) {
        mReactInstanceManager.onResume(this, this);
    }
}
@Override
 public void onBackPressed() {
    if (mReactInstanceManager != null) {
        mReactInstanceManager.onBackPressed();
    } else {
        super.onBackPressed();
    }
}
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
        mReactInstanceManager.showDevOptionsDialog();
        return true;
    }
    return super.onKeyUp(keyCode, event);
}
```
###4. 在项目目录依次运行以下命令，意思是初始化一个RN模块
```
 npm init
 npm install --save react-native
 curl -o .flowconfig https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig
```
上面的代码会创建一个node模块，然后react-native作为npm依赖添加。现在打开新创建的package.json文件然后在scripts字段下添加如下内容：
```
"start": "node node_modules/react-native/local-cli/cli.js start"
```
###5. 复制并粘贴下面的这段代码到你工程根目录下的index.android.js——这是一个简单的React Native应用：
```
'use strict';

var React = require('react-native');
var {
  Text,
  View
} = React;

class MyAwesomeApp extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.hello}>Hello, World</Text>
      </View>
    )
  }
}
var styles = React.StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  hello: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
});

React.AppRegistry.registerComponent('MyAwesomeApp', () => MyAwesomeApp);
```
###6. 启动服务器
```
npm start
```
然后就可以按照常规方式运行APP了，到这里为止是基本的配置。

但是！！！你会遇到很多问题，遇到的话到下面对号入座吧

##问题：
###1. java.lang.UnsatisfiedLinkError: could find DSO to load: libreactnativejni.so
解决：
```
android {    
  ...
  defaultConfig {        
    ...  
    ndk { 
        abiFilters "armeabi-v7a", "x86"     
     }  
  }    
  ...
 packagingOptions {    
    exclude "lib/arm64-v8a/librealm-jni.so" 
   }
...
}
```

###2. Android java.lang.IllegalAccessError Method void android.support.v4.net.ConnectivityManagerCompat

解决：
这个是因为你依赖的react-native版本低了，jCenter上最高只有0.20.1的，但那是很老的版本了

- 第一个办法，将appcompat版本改低到23.0.1
- 第二个办法，使用高版本的react-native

其实在node_modules->react-native->android 下面已经包含了目前最新的版本，直接在配置
```
allprojects {
    repositories {
        mavenLocal()
        jcenter()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$projectDir/../node_modules/react-native/android"
        }
    }
}
```

注意：mavenLocal()一定要放到第一个，不然会报错

###3 make sure your bundle is packaged correctly

1. 在assets下新建出index.android.bundle和index.android.map
2. 之后运行
```
react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res/
```

###4 Conflict with dependency 'com.google.code.findbugs:jsr305'. Resolved versions for app (3.0.1) and test app (2.0.1) differ. See http://g.co/androidstudio/app-test-app-conflict for details.

```
Android{
	  configurations.all {
        resolutionStrategy.force 'com.google.code.findbugs:jsr305:1.3.9'
    }
}
```
###5 undefined is not an object (evaluating 'ReactInternals.ReactCurrentOwner')

1. yarn add react@16.0.0-alpha.12
2. package.json里面自己加 
```
"dependencies": {
    "react": "16.0.0-alpha.12",
    "react-native": "^0.45.1"
  }
```
###6 module appregistry is not a registered

蛋疼的出现了这个问题，不知道为啥，我重新reload之后 就可以正常加载了，前提是你的npm start 一定要是运行成功的，服务器要开着的

最后大家看下我写的Demo吧。参照官网教程随便改的，一份代码在chrom和Android平台都是一样的效果！

"第x条记录"那里是一个listView，可以加载更多。最后那个弹窗是我用ajax请求到的数据，弹出来的。
Android端：

Web：
