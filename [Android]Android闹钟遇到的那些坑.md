
---
title: 「Android」Android闹钟遇到的那些坑
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


## 0x00 做过闹钟的话你才会理解的坑

第一次做闹钟程序是在2012年，那时候android最新版本是2.2，2.3发布在即，做了一个整点提醒的小工具，记得很清楚，主要的问题是锁屏之后闹钟不能准时被唤醒，总会晚那么几秒钟，后来没办法把闹钟提前设置几秒钟。不过那时候环境还好，没有遇到攻克不了的问题，重启也可以唤起闹钟的。

但是随着android版本的进化，开发者节操的丢失，问题就越来越难做了。闹钟明明设置了却不能到来；不再设置的时间到来，晚了好久才到；重启之后闹钟就没了等等。当然，还有好多好多，总之很多东西不按照自己期待的来。

项目上线，踩过之后挑一些做总结。

由于时间关系，项目比较赶，所以没有去考虑闹钟无法及时触发的问题，假设了一个最理想的环境。（更多的信息可以参考下面的文章）


## 0x01 闹钟的创建

这里涉及到不同版本设置闹钟的方法，下面的参考文章中已经提到：

``` java
private void startAlarm(Calendar calendar, PendingIntent pendingIntent) {
   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
       // mManager.setWindow(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), 1000* 5, mFirstPIntent);
       mManager.setExact(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
       // mManager.set(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), mSecondPIntent);
   } else {
       mManager.set(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
   }
}
```


## 0x02 闹钟的取消

有的时候可能需要去取消一个闹钟。我们可以通过两种方式来取消：

- 通过 `PendingIntent.cancel();` 来取消

    ``` java
    /**
     * Cancel a currently active PendingIntent.  Only the original application
     * owning a PendingIntent can cancel it.
     */
    public void cancel() {
        try {
            ActivityManagerNative.getDefault().cancelIntentSender(mTarget);
        } catch (RemoteException e) {
        }
    }
    ```
    
    这种方式只有能拿到 `PendingIntent` 才可以。

- 通过 `AlarmManager.cancel(PendingIntent);` 来取消

    ``` java
    /**
     * Remove any alarms with a matching {@link Intent}.
     * Any alarm, of any type, whose Intent matches this one (as defined by
     * {@link Intent#filterEquals}), will be canceled.
     *
     * @param operation IntentSender which matches a previously added
     * IntentSender.
     *
     * @see #set
     */
    public void cancel(PendingIntent operation) {
        try {
            mService.remove(operation);
        } catch (RemoteException ex) {
        }
    }
    ```


## 0x03 闹钟的查看

又个adb命令可以用来查看当前系统中存在的闹钟：

`adb shell dumpsys alarm`

![这里写图片描述](http://img.blog.csdn.net/20170224183136089?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

不是在所有的设备上都好使，比如在我的魅族设备上可以正常使用，而在另一台 OPPO R7C上就不可以。


## 0xFF 参考

- http://www.jianshu.com/p/1f919c6eeff6

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

