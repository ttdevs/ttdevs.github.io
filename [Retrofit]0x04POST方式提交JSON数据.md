
---
title: 「Retrofit」0x04POST方式提交JSON数据
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


## 0x00 HTTP Method：POST

POST请求再日常的使用中很常见，比如登录，上传数据中使用。之前介绍了GET方式，今天简单介绍下如何使用POST来提交数据。


## 0x01 常用的POST方式

POST方式提交数据再浏览器中的表现主要是使用Form，在客户端中国中的主要表现是提交JSON数据。当然，具体是什么数据格式并不重要，我们可以通过抓包来分析：最终数据都是一样的。

### 使用Model对象

首先新建一个model对象，比如：User，添加常用的熟悉和get／set方法。新建我们的Service：

``` java
@POST("/send")
public Call<ResponseBody> modelPost(@Url String url, @Body User user);
```

测试代码：

``` java
@Test
public void modelPost() throws Exception {
   HttpLoggingInterceptor logging = new HttpLoggingInterceptor();
   logging.setLevel(HttpLoggingInterceptor.Level.BASIC);

   OkHttpClient client = RetrofitManager.getClient(logging);

   Retrofit retrofit = RetrofitManager.getRetrofit(client);
   ExampleService service = retrofit.create(ExampleService.class);

   String url = "http://www.remoteurl.com";
   User user = new User();
   user.setName("ttdevs");
   Call<ResponseBody> example = service.modelPost(url, user);

   final CountDownLatch countDownLatch = new CountDownLatch(1);
   example.enqueue(new Callback<ResponseBody>() {
       @Override
       public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
           try {
               print(response.body().string());
           } catch (IOException e) {
               e.printStackTrace();
           }

           countDownLatch.countDown();
       }

       @Override
       public void onFailure(Call<ResponseBody> call, Throwable t) {
           countDownLatch.countDown();
       }
   });
   countDownLatch.await();
}
```

### 使用RequestBody对象

这里我们来提交一份JSON数据，首先还是再Service中创建一个方法：

``` java
@POST("/send")
public Call<ResponseBody> withRequestBody(@Url String url, @Body RequestBody body);
```

再接着创建我们的请求：

``` java
@Test
public void withRequestBody() throws Exception {
    HttpLoggingInterceptor logging = new HttpLoggingInterceptor();
    logging.setLevel(HttpLoggingInterceptor.Level.BASIC);

    OkHttpClient client = RetrofitManager.getClient(logging);

    Retrofit retrofit = RetrofitManager.getRetrofit(client);
    ExampleService service = retrofit.create(ExampleService.class);

    String url = "http://www.remoteurl.com";

    JSONObject result = new JSONObject();
    try {
        result.put("record", "hello");
    } catch (JSONException e) {
        e.printStackTrace();
    }
    RequestBody body = RequestBody.create(MediaType.parse("application/json"), result.toString());
    Call<ResponseBody> example = service.withRequestBody(url, body);

    final CountDownLatch countDownLatch = new CountDownLatch(1);
    example.enqueue(new Callback<ResponseBody>() {
        @Override
        public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
            try {
                print(response.body().string());
            } catch (IOException e) {
                e.printStackTrace();
            }

            countDownLatch.countDown();
        }

        @Override
        public void onFailure(Call<ResponseBody> call, Throwable t) {
            countDownLatch.countDown();
        }
    });
    countDownLatch.await();
}
```


## 0x02 总结

第一种方法，我们需要为每一个请求的对象创建一个Model，如果你不想创建model则可以选择第二种方式，直接创建一个JSON字符串，然后提交即可。还是相当简答的。

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


