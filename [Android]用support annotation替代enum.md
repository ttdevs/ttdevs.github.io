
---
title: 「Android」用support annotation替代enum
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



>Allocation.MipmapControl, 
Authenticator.RequestorType, 
AvoidXfermode.Mode, 
Bitmap.CompressFormat, 
Bitmap.Config, BlurMaskFilter.Blur,
Canvas.EdgeType, 
Canvas.VertexMode, 
ClientInfoStatus, 
ConsoleMessage.MessageLevel, 
CursorJoiner.Result, 
and 48 others.

简单看了下，Android还是有好多地方使用了。那为什么还不建议使用呢？查了一番，看到google官方有一段关于enum使用的[介绍](1)。既然使用不当可能会导致性能问题，那我们怎么办呢？

这里介绍下Support包中的annotation，使用 `@IntDef` 和 `@StringDef` 来达到我们枚举的效果。由于`@IntDef` 的介绍比较多，这里给个`@StringDef` 的Demo：

``` java
public class MyClient {
    public static final String ONE = "one";
    public static final String RECORD = "record";
    public static final String STATUS = "status";

    @StringDef({ONE, RECORD, STATUS})
    public @interface Client {
    }

    private String mClient;

    public MyClient(@Client String client) {
        mClient = client;
    }
    ...
}
```

这样，在外面创建MyClient对象的时候，构造方法中传入的参数就必须为 `ONE, RECORD, STATUS` 其中之一，从而达到类似enum的效果。


参考
------
1. https://noobcoderblog.wordpress.com/2015/04/12/java-enum-and-android-intdefstringdef-annotation/
2. [The price of ENUMs](https://www.youtube.com/watch?v=Hzs6OBcvNQE)
3. http://tools.android.com/tech-docs/support-annotations
4. http://developer.android.com/reference/java/lang/Enum.html
5. http://stackoverflow.com/questions/29183904/should-i-strictly-avoid-using-enums-on-android

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

