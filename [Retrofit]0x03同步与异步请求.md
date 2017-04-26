
## 0x00 请求的形式

我们正常的网络请求有两种形式：同步方式和异步方式。所谓同步方式，是指我们在在发出网络请求之后当前线程呗阻塞，直到请求的结果（成功或者失败）到来，才继续向下执行。所谓异步，是指我们的网络请求发出之后，不必等待请求结果的到来，就可以去做其他的事情，当请求结果到来时，我们在做处理结果的动作。当时无论是同步还是异步，最终都是同步请求。


## 0x01 同步请求

Retrofit的同步请求比较简单，直接调用 `Call` 的execute方法即可。这个时候我们会拿到请求的结果。

``` java
   @Test
    public void synchronousRequest() throws Exception {
        Retrofit retrofit = RetrofitManager.getRetrofit();
        GitHubService service = retrofit.create(GitHubService.class);
        Call<User> example = service.userInfo("ttdevs");
        Response<User> response = example.execute();
        print(response.body().getName());
    }
```


## 0x02 异步请求

之前文章的请求都是异步方式，在请求的回调中我们会拿到请求结果，接下来 处理这个结果即可。

``` java
@Test
    public void requestHeader() throws Exception {
        OkHttpClient client = RetrofitManager.getClient(new Interceptor() {
            @Override
            public okhttp3.Response intercept(Chain chain) throws IOException {
                Request original = chain.request();
                Request request = original.newBuilder()
                        .header("Time-Zone", "Asia/Shanghai") //https://developer.github.com/v3/#timezones
                        .header("user_key", "I_am_user_key")
                        .method(original.method(), original.body())
                        .build();
                okhttp3.Response response = chain.proceed(request);
                return response;
            }
        });
        Retrofit retrofit = RetrofitManager.getRetrofit(client);
        ExampleService service = retrofit.create(ExampleService.class);

        String url = "http://www.weather.com.cn/adat/sk/101020100.html";
        Call<ResponseBody> example = service.requestWithHeader(url);

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


## 总结

由于同步请求会阻塞当前线程，android的设计逻辑不能在主线程中发起网络请求（可能会导致ANR），所以我们比较常见的都是异步形式的网络请求。但是，有时候我们在工作线程中处理一些网络请求的时候，可以接受阻塞的情况（这个时候使用回调往往会使问题复杂化），那么我们就可以使用同步的方式来发起网络请求，比如，在使用Rxjava封装某个网络功能的逻辑是（如图片上传）我们就可以同步形式的网络请求。

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

