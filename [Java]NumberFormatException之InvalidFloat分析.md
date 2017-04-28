
---
title: 「Java」NumberFormatException之InvalidFloat分析
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Java
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x00 问题

新版本上线，4天时间一个错误出现一百多次，如下：

``` java
java.lang.NumberFormatException: Invalid float: "55,4"
	at java.lang.StringToReal.invalidReal(StringToReal.java:63)
	at java.lang.StringToReal.initialParse(StringToReal.java:164)
	at java.lang.StringToReal.parseFloat(StringToReal.java:323)
	at java.lang.Float.parseFloat(Float.java:306)
```


## 0x01 分析

发现此问题后感觉很熟悉，以前应该是见过，但是忘记怎么解决的了，没办法，踩过的坑再踩一遍。涉及的代码如下：

``` java
public class OnePreference{
    ...
    public static void setLatestWeight(float weight) {
       getInstance(MyApplication.getContext()).putFloat(PREFS_LATEST_WEIGHT, Float.parseFloat(NumberUtils.formatFloatWithOneDot(weight)));
    }
    ...
}

public class NumberUtils {
    ...
    public static String formatFloatWithOneDot(float number) {
        number = Math.round(number * 10) / 10f;
        return String.format("%.1f", number);
    }
    ...
}
```

- 确认问题
    
    android无法复现，iOS后台未报此错误
    
- 怀疑是数据的问题
    
    这是一个记录体重的功能，新版体重输入已经做了限制，所以怀疑可能是老数据的问题。因此找后端的同事降此数据做处理：如果此字段无法解析为数字则过滤掉。
    但是遗憾的，后端发布之后问题依然存在。
    
- 分析代码

    `setLatestWeight(float weight)` 这个传入的是float，所以之前的假设是错误的 —— 如果是数据的问题，那根本走不到方法里。
    这样的话那就是 `formatFloatWithOneDot(float number)` 这个方法的问题，但是想了下之前都有这么写过 —— 将float类型格式化为一位小数的字符串，应该没问题啊。那会是怎样呢？可能很自然会想到国际化的问题。遂作了一个简单的测试：
    
    ``` java
public class ExampleUnitTest {

    @Test
    public void testStringLocale() throws Exception {
        Locale[] locales = new Locale[]{
                Locale.CANADA,
                Locale.CANADA_FRENCH,
                Locale.CHINESE,
                Locale.ENGLISH,
                Locale.FRANCE,
                Locale.GERMAN,
                Locale.GERMANY,
                Locale.ITALIAN,
                Locale.ITALY,
                Locale.JAPAN,
                Locale.JAPANESE,
                Locale.KOREA,
                Locale.KOREAN,
                Locale.PRC,
                Locale.ROOT,
                Locale.SIMPLIFIED_CHINESE,
                Locale.TAIWAN,
                Locale.TRADITIONAL_CHINESE,
                Locale.UK,
                Locale.US
        };

        String weightString = null;
        for (Locale locale : locales) {
            try {
                weightString = formatFloatWithOneDot(locale, 55.4f);
                float weight = Float.parseFloat(weightString);
            } catch (NumberFormatException e) {
                System.out.println(locale + ">>>>>" + weightString + ">>>>>>>>>> error");
                continue;
            }
            System.out.println(locale + ">>>>>" + weightString);
        }
    }

    public static String formatFloatWithOneDot(Locale locale, float number) {
        number = Math.round(number * 10) / 10f;
        return String.format(locale, "%.1f", number);
    }
}
    ```
    
    输出结果如下：
    
    ``` shell
    en_CA>>>>>55.4
    fr_CA>>>>>55,4>>>>>>>>>> error
    zh>>>>>55.4
    en>>>>>55.4
    fr_FR>>>>>55,4>>>>>>>>>> error
    de>>>>>55,4>>>>>>>>>> error
    de_DE>>>>>55,4>>>>>>>>>> error
    it>>>>>55,4>>>>>>>>>> error
    it_IT>>>>>55,4>>>>>>>>>> error
    ja_JP>>>>>55.4
    ja>>>>>55.4
    ko_KR>>>>>55.4
    ko>>>>>55.4
    zh_CN>>>>>55.4
    >>>>>55.4
    zh_CN>>>>>55.4
    zh_TW>>>>>55.4
    zh_TW>>>>>55.4
    en_GB>>>>>55.4
    en_US>>>>>55.4
    
    Process finished with exit code 0
    ```
    
    我们可以看到，在不同的语言下 `String.format(locale, "%.1f", number)` 格式化的结果是不一样的，55.4可能会被格式化为55,4，然后就出现了上面的错误。将我们的测试机语言设置成出错的语言，问题可以复现。
    
    再来看下后台的错误：
    
    ![错误信息]()
       
    - 大部分是华为的设备，然后是三星，索尼
    - 各个版本都有，从4.4到最新的7.0
    - 不同的渠道都有，主要的渠道来自华为
        
    这些信息也从侧面反映了问题的原因。看来华为的国际化之路还是很屌的。


## 0x02 解决

- 由于我们并没有做国际化的适配，因此这里改起来还是很方便的，直接给 `String.format()` 方法指定Locale信息：

    `String.format(Locale.ENGLISH, "%.1f", number);`
    
    or
    
    `String.format(Locale.CHINA, "%.1f", number);`        

- NumberFormat(DecimalFormat)


![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

