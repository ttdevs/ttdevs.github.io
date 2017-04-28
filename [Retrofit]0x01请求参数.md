
---
title: 「Retrofit」0x01请求参数
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


## 0x00 请求参数

常见的Http请求，除了指定的请求地址，很多时候我们还需要加上一些请求参数，这些参数可能是固定的，也可能是动态添加的。在Retrofit中，我们该如何处理呢？这里以GET方式中添加请求参数为例。


## 0x01 固定请求参数

这里说的固定请求参数是指每次请求我们都在Url中添加相同的请求的参数。这种场景多出现在需要认证的情况，比如添加 Token 字段等。

在之前的文章中，我们有提到在构建OkHttpClient对象的时候使用过 `Interceptor` ，在 `Interceptor` 的逻辑中，我们会重新创建一个 `Request` 对象，在这个对象中我们可以传入一个 `HttpUrl` ，在这个对象中，我们可以传入一些需要的请求参数。通过之前对 `Interceptor` 了解，我们应该知道，这里做的操作是对所有的请求都有效的。这样就可以完成我们为每个请求添加固定请求参数的目的。代码如下：

``` java
OkHttpClient client = new OkHttpClient.Builder()
                .addInterceptor(new Interceptor() {
                    @Override
                    public okhttp3.Response intercept(Chain chain) throws IOException {
                        Request original = chain.request();

                        HttpUrl originalHttpUrl = original.url();

                        HttpUrl url = originalHttpUrl
                                .newBuilder()
                                .addQueryParameter("token", "I_am_user_token")
                                .addQueryParameter("user_key", "I_am_user_user_key")
                                .build();

                        Request request = original
                                .newBuilder()
                                .url(url)
                                .build();

                        okhttp3.Response response = chain.proceed(request);
                        return response;
                    }
                })
                .build();
```


## 0x02 动态请求参数

这个指我们在不同的请求中添加不同过的请求参数。比如，在请求用户信息的接口中我们可能需要传入用户ID，在请求记录的接口中我们需要传入起止时间等。由于这样的参数是针对接口的，因此我们就需要在每一个接口做中处理。

- @Query 添加一个请求参数

	``` java
    // http://119.29.29.29/d?dn=ttdevs.vicp.com
    @GET("/d")
    public Call<ResponseBody> singleParams(@Query("dn") String domain);
	```
	
	通过上面的方式，我们发出的请求就是：
	
	`http://119.29.29.29/d?dn={domain}`

- @QueryMap 添加一组请求参数

	``` java
	@GET("/record")
 	public Call<ResponseBody> multiParams(@QueryMap(encoded = true) Map<String, String> options);
	```

	上面我们发出的请求是：
	
	`http://ttdevs.vicp.net/record?p1=v1&p2=v2`
	
	这个这个例子中还可以看到，我们使用了一个参数：`encoded = true`，这个参数的意思是，对我们的参数进行URL
	Encode。

	
## 0x03 总结

通过上面的介绍，我们可以了解，如果添加我们的请求参数。当需要给每一个请求都添加相同参数时该如何封装，把Retrofit打造成更适合我们使用的工具。

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


