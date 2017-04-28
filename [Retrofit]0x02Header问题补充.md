
---
title: 「Retrofit」0x02Header问题补充
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Retrofit
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x00 Retrofit Header

上一篇中我们介绍Retrofit的一个简单的Demo和添加Header方法，这一篇在补充一下Header的另外几种处理方法。


## 0x01 Header

  使用OkHTTP的Interceptor来处理Header信息，这种方法是需要我们在封装Retrofit的时候考虑的。但是有时候我们还希望能灵活的控制每一个请求的Header信息，接下来的几种方法满足我们这样的要求。他们都是在各自的Service方法上添加注解来达到添加Header的目的。

### 添加单个Header字段

``` java
@Headers("User-Agent: Your-App-Name")
@GET
public Call<ResponseBody> weatherReport(String cityCode);
```

### 添加多个Header字段

``` java
@Headers({
       "Accept: application/json",
       "User-Agent: ttdevs"
})
@GET
public Call<ResponseBody> weatherReport(String cityCode);
```

### 使用Map添加单个Header字段

``` java
@GET
public Call<ResponseBody> requestWithHeaderMap(@HeaderMap Map<String, String> header);
```
  
源码参考：[ttdevs](https://github.com/ttdevs/android/tree/master/modules/retrofit)


## 0xFF 参考

1. https://github.com/square/retrofit
2. http://square.github.io/retrofit/
3. https://futurestud.io/

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

