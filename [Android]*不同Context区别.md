## 0x01 Dialog

代码：

``` java
AlertDialog dialog = new AlertDialog
    .Builder(context)
    .setTitle(title)
    .setCancelable(false)
    .setPositiveButton("确定", listener)
    .create();
dialog.show();
```

错误信息：

``` java
android.view.WindowManager$BadTokenException: Unable to add window -- token null is not for an 
	at android.view.ViewRootImpl.setView(ViewRootImpl.java:589)
	at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:326)
	at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:224)
	at android.view.WindowManagerImpl$CompatModeWrapper.addView(WindowManagerImpl.java:149)
	at android.app.Dialog.show(Dialog.java:293)
	at com.ttdevs.easysecuritysmartbar.StartHandler.handleMessage(StartHandler.java:39)
	at android.os.Handler.dispatchMessage(Handler.java:99)
	at android.os.Looper.loop(Looper.java:137)
	at android.app.ActivityThread.main(ActivityThread.java:4866)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:511)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:786)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:553)
	at dalvik.system.NativeStart.main(Native Method)
```

其中context是通 `getApplicationContext()` 获取的，将其换成 `Activity` 问题解决。


## 0xFF 参考

1. http://blog.csdn.net/lmj623565791/article/details/40481055



