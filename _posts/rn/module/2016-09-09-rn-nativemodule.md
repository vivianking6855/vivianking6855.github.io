---
layout: post
title: ReactNative 学习笔记 Android Native Module
date: 2016-06-30
excerpt: "ReactNative 学习笔记 Android Native Module"
tags: [ReactNative,学习笔记]
comments: true
---

## Android Native Moudule

在RN控件或功能无法满足需求时，需要Native Module来帮忙，需要平台API。

使用场景：React Native还不支持某个你需要的原生特性，你可以自己实现该特性的封装。

如何封装这些RN没有的模块呢？现有步骤介绍和Sample Code

## Argument Types
@ReactMethod RN参数跟JS参数的对比map，我们demo中用到了String，WritableMap，Promise，Callback，int 

Boolean -> Bool
Integer -> Number
Double -> Number
Float -> Number
String -> String
Callback -> function
ReadableMap -> Object
ReadableArray -> Array

Promise -> Promise
WritableMap -> Object


## Toast 模块例子
我们希望可以从Javascript发起一个Toast消息（Android中的一种会在屏幕下方弹出、保持一段时间的消息通知）

1. 创建一个原生模块 (UserNativeModule.java)

        核心Code:
    
        private static final String DURATION_SHORT_KEY = "SHORT";
        private static final String DURATION_LONG_KEY = "LONG";
    
        public UserNativeModule(ReactApplicationContext reactContext) {
            super(reactContext);
        }
    
        @Override
        public String getName() {
            return "UserNativeModule";
        }
    
        @Override
        public Map<String, Object> getConstants() {
            final Map<String, Object> constants = new HashMap<>();
            constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
            constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
            return constants;
        }
    
        /**
         * @param message the text to show in toast
         * @param duration toast show time
         */
        @ReactMethod
        public void showToast(String message, int duration) {
            Toast.makeText(getReactApplicationContext(), message, duration).show();
        }
2. 注册模块 (UserReactPackage.java)

        核心Code：
        @Override
        public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
            List<NativeModule> modules = new ArrayList<>();
    
            modules.add(new UserNativeModule(reactContext));
    
            return modules;
        }


3. add to getPackages (MainApplication.java)

        核心Code:
        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
                    new UserReactPackage()
            );
        }
4. JS中使用封装的方法

把原生模块封装成一个JavaScript模块，可以省下了每次都从NativeModules中获取对应模块的步骤。

    UserNativeModule.js 核心Code
        
        'use strict';
        
        /**
         * This exposes the native UserNativeModules module as a JS module. 
         * This has a some function of UserNativeModules. such as  'showToast', 'showActivity'
         * for detail parametoers, please check UserNativeModule.java in android folder
         * 
         * @param message the text to show in toast
         * @param duration toast show time
           @ReactMethod
           public void showToast(String message, int duration) {...}
         */
        let { NativeModules } = require('react-native');
        
        module.exports = NativeModules.UserNativeModule;


    JS中使用核心Code
    
        import UserNativeModule from './common/UserNativeModule';
        
        export default class Root extends Component {
        
          onPress() {
            UserNativeModule.showToast('Hi ASUS', UserNativeModule.SHORT);
          }
        
          render() {
            return (
              <View>
                <Text style={styles.text} onPress={this.onPress.bind(this) }>Native Toast</Text>
              </View>
            )
          }
        }
        
        const styles = StyleSheet.create({
          text: {
            flex: 1,
            fontSize: 20,
            margin:20,
          }
        });


## Activity例子
我们希望可以从Javascript叫起Android端Activity

1. 原生模块UserNativeModule.java中添加方法

        核心Code:
    
        /**
         * @param module   activity id that which activity you want to call up
         * @param map
         * @param callback
         */
        @ReactMethod
        public void startActivity(int module, final ReadableMap map, Callback callback) {
            mCallback = callback;
            Activity currentActivity = getCurrentActivity();
            switch (module) {
                case ACTIVITY_TEST_ID:
                    startTestActivity(map, currentActivity);
            }
        }
    
        /**
         * @param map
         * @param activity
         */
        public void startTestActivity(final ReadableMap map, Activity activity) {
            Intent it = new Intent(getReactApplicationContext(), TestActivity.class);
            it.putExtra("status", map.getInt("status"));
            it.putExtra("text", map.getString("text"));
            activity.startActivityForResult(it, ACTIVITY_TEST_ID);
        }
    
        @Override
        public void onActivityResult(final int requestCode, final int resultCode, final Intent intent) {
            switch (requestCode) {
                case ACTIVITY_TEST_ID:
                    String result = intent.getStringExtra("result");
                    if (mCallback != null) {
                        mCallback.invoke(result);
                    }
                    break;
            }
        }
    
2. 实现Android TestActivity
   Android正常步骤创建Activity，这里不多赘述，Code放在[这里](https://github.com/vivianking6855/ReactNativeProject/blob/master/Advanced/nativemodules/android/app/src/main/java/com/nativemodules/TestActivity.java)
   

3. JS中使用

    核心Code
    
        const map = {
          'status': 0,
          'text': 'ASUS from RN',
        };
        UserNativeModule.startActivity(UserNativeModule.ACTIVITY_TEST, map, (result) => {
          ToastAndroid.show(result, ToastAndroid.SHORT);
        });
        
        
## Promise
 JS的异步变成模块。 RN也直接传入Promise作为参数。下面以获取Android app版号方法为例：
 Android核心Code
    @ReactMethod
    public void getPackageVersion(final Promise promise) {
        String versionName = "1.0";
        int versionCode = 1;
        try {
            PackageManager packageManager = getReactApplicationContext().getPackageManager();
            PackageInfo info = packageManager.getPackageInfo(getReactApplicationContext().getPackageName(), 0);
            versionName = info.versionName;
            versionCode = info.versionCode;

            WritableMap ret = Arguments.createMap();
            ret.putString("versionName", versionName);
            ret.putInt("versionCode", versionCode);
            promise.resolve(ret);
        } catch (Exception ex) {
            Log.d(TAG, "getPackageVersion:", ex);
            promise.reject(ex);
        }
    }
    
  JS使用
  
        UserNativeModule.getPackageVersion()
          .then(result => {
            let version = JSON.stringify(result);
            ToastAndroid.show('Version : ' + version, ToastAndroid.LONG);
          })
          .catch(result => {
            ToastAndroid.show('exception : ' + result);
          });
          
## RN android端调用JS模块 - RCTDeviceEventEmitter 组件
RCTDeviceEventEmitter通讯组件不仅仅可以用于[组件全局事件交互](http://vivianking6855.github.io/rn-Community-post/)，还可以用做android调用JS模块使用

Android核心Code

    TestActivity.java
    
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.jsBtn:
                sendEvent();
                break;
        }
    }

    private void sendEvent() {
        WritableMap params = Arguments.createMap();
        params.putString("text", "event from android");
        UserNativeModule.sendEvent(EVENT_NAME, params);
    }
    
    UserNativeModule.java
    
    public static void sendEvent(String eventName, WritableMap params) {
        sUserNativeModule.mReactContext
                .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
                .emit(eventName, params);
    }
    

JS使用

    import {
      DeviceEventEmitter
    } from 'react-native';

      componentWillMount() {
        DeviceEventEmitter.addListener('TEST', this.handleTest.bind(this));
      }
    
      componentWillUnmount() {
        DeviceEventEmitter.removeListener('TEST');
      }
    
      handleTest(param) {
        ToastAndroid.show('JS rev event: ' + JSON.stringify(param), ToastAndroid.SHORT);
      }


PS: ReactContext在其他Activity中不太好获取，因此我将方法放在UserNativeModule.java中。同时在UserNativeModule中设置singleton给外部使用


[Demo下载地址](https://github.com/vivianking6855/ReactNativeProject/tree/master/Advanced/nativemodules)


> [FB Native Modules](https://facebook.github.io/react-native/docs/native-modules-android.html)

> [RN 中文网 Native Modules](http://reactnative.cn/docs/0.26/native-modules-android.html#content)