
---
title: 「Android」对App进行代码混淆
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



接到一个新的任务，对现有项目进行代码混淆。之前对混淆有过一些了解，但是不够详细和完整，知道有些东西混淆起来还是比较棘手的。不过幸好目前的项目不是太复杂（针对混淆这块来说），提前完成～～ 现总结之。


## 0x01 第一部分

介绍下操作流程（eclipse）：

- 打开混淆器

    找到项目根目录下的 `project.properties` 文件
    `#proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt` 
    将上面这行前的 `#` 删除即可

- 修改混淆配置文件

    找到项目根目录下的 `proguard-project.txt` 文件，修改其中代码，这部分是最关键

- 保存相关文件供以后出错时使用

    主要有导出的apk文件、项目根目录下的proguard目录下的文件（主要的是mapping.txt）和项目源码

- 项目运行过程出错处理

    根据错误信息和第3步中保存的mapping定位错误位置
    

知道这些之后，我们对其进行展开。

打开eclipse然后新建一个项目，默认会创建 `proguard-project.txt` 和`project.properties`。编写我们的代码，修改 `proguard-project.txt` ：

`#proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt`

`:proguard-project.txt` 

将这行前的 `#` 删除，最后导出即可实现对代码的混淆（即使我们没有去编写proguard-
project.txt中的内容）。下面是我的测试代码：

``` java
public class MainActivity extends Activity {

    private String mName;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mName = "ttdevs";

        getString(mName);
        setName(mName);
        showDialog();
        // testError();
    }

    public String getString(String name) {
        return "hello " + name;
    }

    public void setName(String name) {
        System.out.println("I'm " + name);
    }

    private void showDialog() {
        new Handler().postDelayed(new Runnable() {

            @Override
            public void run() {
                ScoreAlertDialog.showDialog(MainActivity.this);
            }
        }, 2000);
    }

    public static class ScoreAlertDialog {

        public static void showDialog(final Activity activity) {
            if (activity.isFinishing()) {
                return;
            }
            try {
                AlertDialog.Builder builder = new AlertDialog.Builder(activity);
                builder.setTitle("alert_title");
                builder.setNegativeButton("cancel", null);
                builder.setPositiveButton("submit", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        try {
                            Toast.makeText(activity, "Welcome", Toast.LENGTH_LONG).show();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                });
                builder.show();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    private void testError() {
        try {
            int error = 1 / 0;
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

打包，反编译，最后我们得到如下的代码：

![](http://img.blog.csdn.net/20140928202555529?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

分析上面的代码我们会发现，自定义的方法名都被替换成无特殊意义的短字母，而activity的onCreate()方法却没变；最后一个testError()方法由于我们没有调用也被剔除掉了。这些就是默认的混淆处理策略。看到这里，感觉混淆还是小case的哈～～

继续往下，我们将注销的testError()打开，打包运行这个时候会报错，错误信息如下：

``` java
java.lang.ArithmeticException: divide by zero
	at com.ttdevs.proguard.MainActivity.b(Unknown Source)
	at com.ttdevs.proguard.MainActivity.onCreate(Unknown Source)
	at android.app.Activity.performCreate(Activity.java:4531)
	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1071)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2150)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2229)
	at android.app.ActivityThread.access$600(ActivityThread.java:139)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1261)
	at android.os.Handler.dispatchMessage(Handler.java:99)
	at android.os.Looper.loop(Looper.java:154)
	at android.app.ActivityThread.main(ActivityThread.java:4945)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:511)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:784)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:551)
	at dalvik.system.NativeStart.main(Native Method)
```

由于这个例子比较简单，很容易看出来是何地方出了问题，不过还是可以用来说明我们想表达的问题：如何还原混淆后的代码的错误信息？
为了达到这个目的，我们需要三个文件:

- android-sdk-windows\tools\proguard\bin\retrace.bat
- mapping.txt
- log.info（上面的错误信息）

然后执行下面的命令（window系统）

`retrace.bat mapping.txt log.txt`

![](http://img.blog.csdn.net/20140928221717551?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

从上图中可以很清楚的看到错误日志中的b()方法为我们实际代码中的setName()方法。

这里需要注意的是每次导出apk都会在项目中目录下的proguard文件夹下生成一个对应的mapping.txt 文件，所以对于每个apk我们都需要保存与之对应的mapping.txt文件。至此整个混淆的流程介绍完毕。

### 参考

- 官方文档

[http://developer.android.com/tools/help/proguard.html](http://developer.android.com/tools/help/proguard.html)

- 官方文档的翻译

[http://www.cnblogs.com/over140/archive/2011/04/22/2024528.html](http://www.cnblogs.com/over140/archive/2011/04/22/2024528.html)

（本想自己去翻一个，结果发现很久以前农民伯伯已经翻译，在此直接引用并感谢之）


## 0x02 第二部分

第一部分讲了如何操作，参照官方文档，基本都会掌握。剩下的也是最难的就是 `proguard-project.txt` 文件的编写。对于这部分，两种处理策略：自己编写
和使用别人写好的。先说如何使用别人写好的，我们引用的第三方库无论开源还是闭源如有特殊情况我们都可以在他的User Guide中找到混淆代码的配置，如我们引用的大名鼎鼎的 [guillep PullToRefresh](https://github.com/guillep/PullToRefresh) ，我们可以在[他的文档](https://github.com/guillep/PullToRefresh/blob/master/PullToRefresh/proguard.cfg) 中找到如下的代码：

``` java
-optimizationpasses 5
-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-dontpreverify
-verbose
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*
    
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class com.android.vending.licensing.ILicensingService
    
-keepclasseswithmembernames class * {
   native <methods>;
}
    
-keepclasseswithmembernames class * {
   public <init>(android.content.Context, android.util.AttributeSet);
}
    
-keepclasseswithmembernames class * {
   public <init>(android.content.Context, android.util.AttributeSet, int);
}
    
-keepclassmembers enum * {
   public static **[] values();
   public static ** valueOf(java.lang.String);
}
    
-keep class * implements android.os.Parcelable {
 public static final android.os.Parcelable$Creator *;
}
```

有了这部分代码我们就可以直接copy插入我们的项目中即可。这种方式还是copy式的。那下面我们举个小例子看看如何自己写代码控制是否混淆。还是用第一部分的例子，我们在这个项目的 proguard-project.txt 文件中（之前为空）加入如下几行（proguard-project.txt中“#”代表注释）：

``` java
# -keep public class com.ttdevs.proguard.** { *; }
# -keepclasseswithmembers public class com.ttdevs.proguard.** { *; }
    
-keep public class com.ttdevs.proguard.MainActivity {
java.lang.String getString(java.lang.String);
}
```

然后我们导出apk再反编译，得到如下代码：

![](http://img.blog.csdn.net/20140928225529585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
  
和之前的对比，我们发现其中的getString方法没有被混淆。没错，上面 proguard-project.txt的意思就是保持MainActivity的getString()方法不要被混淆。大家也可以试试上述混淆代码中被注释的两行分别是什么效果。

讲到这里已经开始涉及ProGuard的核心部分了，剩下的就是研读 [ProGuard的文档](http://proguard.sourceforge.net) ，掌握的他的语法并使用之。本想找一个完整的ProGuard的翻译文档，但是找了N久没有发现一个，而且连零零散散的翻译也非常的少，最近时间很紧，加之能力有限，想翻译一下常用的几个命令也是很困，所以细读的想法只能暂时往后推了。这里先简单介绍下keep命令：

[-keep [,modifier,...] class_specification](http://proguard.sourceforge.net/manual/usage.html#keep)

在你的代码中指定作为切入点而被保留的类或者类的成员（属性和方法）。例如，为了保持一个应用，你可以指定主类和他的main方法。为了处理一个库，你需要详细说明他的public访问的元素。  

[另外还有keep的简单概述](http://proguard.sourceforge.net/manual/usage.html#keepoverview) 和 [语法中规范](http://proguard.sourceforge.net/manual/usage.html#classspecification) 。

Class Specification中会告诉你如何表示构造方法，属性和方法，`*` 与`**`的区别等等。比如 `*` 表示匹配任何的类名但是不包括包的分隔符，而 `**` 则是匹配任何的类名并且包括任意数量的包分隔符，因此上面我们注释掉的代码意思如下：

- 第一行 

    保持com.ttdevs.proguard下的所有类和子包下的类的所有方法都不混淆
- 第二行

    保持com.ttdevs.proguard下的所有类和子包下的类的所有方法和成员变量都不混淆。

// TODO 细节还有很多，比如-libraryjars、-dontwarn、-keepattributes等等，这些待续吧

### 参考

- ProGuard 5.0

    [ProGuard 5.0](http://proguard.sourceforge.net/) 
    
- ProGuard 4.7

    [ProGuard 4.7](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html) 

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)
  

