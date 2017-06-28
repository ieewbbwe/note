1
```
compile 'com.facebook.react:react-native:0.45.1'
```
2
```
<uses-permission android:name="android.permission.INTERNET" />
```
3
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
4
```
 npm init
 npm install --save react-native
 curl -o .flowconfig https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig
```
上面的代码会创建一个node模块，然后react-native作为npm依赖添加。现在打开新创建的package.json文件然后在scripts字段下添加如下内容：
```
"start": "node node_modules/react-native/local-cli/cli.js start"
```
5
复制并粘贴下面的这段代码到你工程根目录下的index.android.js——这是一个简单的React Native应用：
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
6
```
npm start
```
##问题：
1. java.lang.UnsatisfiedLinkError: could find DSO to load: libreactnativejni.so
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

2. Android java.lang.IllegalAccessError Method void android.support.v4.net.ConnectivityManagerCompat

解决：这个是因为你依赖的react-native版本低了，jCenter上最高只有0.20.1的，但那是很老的版本了
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

3 make sure your bundle is packaged correctly

在assets下新建出index.android.bundle和index.android.map
之后运行

react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res/

4 Conflict with dependency 'com.google.code.findbugs:jsr305'. Resolved versions for app (3.0.1) and test app (2.0.1) differ. See http://g.co/androidstudio/app-test-app-conflict for details.

```
Android{
	  configurations.all {
        resolutionStrategy.force 'com.google.code.findbugs:jsr305:1.3.9'
    }
}
```
