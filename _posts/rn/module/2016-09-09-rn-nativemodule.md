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



> [FB Native Modules](https://facebook.github.io/react-native/docs/native-modules-android.html)

> [RN 中文网 Native Modules](http://reactnative.cn/docs/0.26/native-modules-android.html#content)