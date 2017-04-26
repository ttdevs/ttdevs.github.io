
## 0x00 问题

通过Data Binding给Button设置背景无效。
具体表现为给Button设置不同的背景图片，但是无论怎样背景只会显示不同的颜色，而不是显示期望的图片。


## 0x01 分析

由于对Data Binding不是很熟悉，所以很奇怪为什么。不管怎样先打印下这个资源的值，发现在布局文件中打印资源值是有的。根据经验，设置不同的背景图片，背景会显示不同的颜色，那这个值可能被解析成了颜色值。


## 0x02 解决

在网上搜了一下（容我忘记了具体网址），需要通过不同的方式来设置背景：
之前的：

``` xml
<Button
      android:layout_width="200dp"
      android:layout_height="200dp"
      android:gravity="center"
      android:onClick="@{(view) -> handler.onStartClick(view)}"
      android:text="@string/button_start"
      android:textColor="@color/blueColor"
      android:textSize="@dimen/global_font_large"
      android:background="@{handler.startButtonBg}"
      tools:background="@drawable/btn_main_circle_start" />
```

修改之后：

``` xml
<Button
      android:layout_width="200dp"
      android:layout_height="200dp"
      android:gravity="center"
      android:onClick="@{(view) -> handler.onStartClick(view)}"
      android:text="@string/button_start"
      android:textColor="@color/blueColor"
      android:textSize="@dimen/global_font_large"
      app:backgroundResource="@{handler.startButtonBg}"
      tools:background="@drawable/btn_main_circle_start" />
```

也就是将之前的 `android:background` 换成 `app:backgroundResource`


## 0x03 总结

没啥总结的，Data Binding 还有很多等待探究。

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


