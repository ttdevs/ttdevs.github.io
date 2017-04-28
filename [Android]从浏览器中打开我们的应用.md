
---
title: 「Android」从浏览器中打开我们的应用
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Android
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x01 需求

有时候你会发现，用Android系统自带的浏览器（chrome）打开一个web页面，如果安装了相应的App，就会自动打开这个App并进入具体的界面中，比如手机上打开：

`https://www.zhihu.com/openinapp_instruction?app-id=432274380&app-argument=zhihu://questions/24122524`

如果我们安装了知乎手机客户端，这个时候会自动打开知乎手机客户端，并且进入问题展示界面。还比如网易云音乐，今日头条等等App都是支持这个功能的。之前我们自家App的分享都是打开一个App介绍的界面，即使安装了手机客户端，也不会自动打开手机客户端。为了追求更好的体验，我们需要实现这个功能。


## 0x01 Chrome浏览器

研究这个功能，还是查阅了不少资料，比如 app link、deep link、chrome browser等等，如果紧紧是实现上面介绍的功能，还是 so easy的，仅仅需要了解 `Chrome` 浏览器 创建 Intent 的方法即可。

Chrome构造一个 Intent 的基本语法如下：

``` html
intent:
   HOST/URI-path // Optional host 
   #Intent; 
      package=[string]; 
      action=[string]; 
      category=[string]; 
      component=[string]; 
      scheme=[string]; 
   end; 
```

根据上面的语法，我们来看一个demo：

``` html
intent:
   //article/51348656
   #Intent; 
      package=com.ttdevs.android; 
      scheme=ttdevs; 
   end; 
```

可以使用下面的 html 代码触发这个事件：

``` html
<a href="intent://article/51348656/#Intent;scheme=ttdevs;package=com.ttdevs.android;end">ttdevs</a>
```

另外还有个参数：

``` html
S.browser_fallback_url=[encoded_full_url]
```

介绍如下（不过我没搞明白）：
>When an intent could not be resolved, or an external application could not be launched, then the user will be redirected to the fallback URL if it was given.
Some example cases where Chrome does not launch an external application are as follows:
- The intent could not be resolved, i.e., no app can handle the intent.
- JavaScript timer tried to open an application without user gesture.
Note that S.<name> is a way to define string extras. S.browser_fallback_url was chosen for backward compatibility, but the target app won’t see browser_fallback_url value as Chrome removes it.


## 0x02 App内处理浏览器发来的Intent

通过上面的介绍，如果我们点击链接，就会触发一个 Intent，有了Intent，我们还需要来处理它。在 `AndroidManifest.xml` 中相应的 Activity 中的配置中添加如下filter：

``` xml
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>

    <data android:host="article" android:pathPattern="/.*" android:scheme="ttdevs"/>
</intent-filter>
```

同时在Activity中处理Intent：

``` java
public class RouterActivity extends BaseActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_router);

        handleIntent(getIntent());
    }

    private void handleIntent(Intent intent) {
        String action = intent.getAction();
        String data = intent.getDataString();
        LogUtils.debug(action);
        LogUtils.debug(data);
    }
}
```

运行上面的代码，同时用系统浏览器打开上面的html，点击链接，即可调起我们的app。或者也可以通过下面的adb命令触发：

``` shell
adb shell am start \
-a android.intent.action.VIEW \
-c android.intent.category.BROWSABLE \
-d "ttdevs://article/12345" com.ttdevs.android
```

在我的设备（魅族MX4 Pro）上，如果没有安装处理这个 Intent 的App，则会打开系统自带的应用市场。

对于一个提供查询食物数据的App，如果我们分享出去的食物信息带上了相关的参数，在用户点击分享链接时，如果安装了我们的App则可以跳到App中展示食物的界面，如果没有安装，则会跳到应用市场下载，很好的逻辑～～


## 0xFF 参考

- https://developer.chrome.com/multidevice/android/intents

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


