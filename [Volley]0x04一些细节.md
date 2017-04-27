#  [ android网络开源框架volley(五)——volley的一些细节 ](/ttdevs/article/details/40260433)

最近又把volley拿出来整理了下。之前没有遇到过的一些小问题又来了，在此记录下：


##  1、HttpUrlConnection DELETE 方式无法添加body的问题：java.net.ProtocolException:
DELETE does not support writing

这个可以算是一个系统级的bug，为什么这么说，请看这里，这个问题在java8中才得以解决。没办法直接过去，咱就绕过去。查看HttpUrlConnection
，我们发现他是一个抽象类，因此可以试试能不能通过它的其他实现来达到我们的目的。最终我们决定使用 [ okhttp
](https://github.com/square/okhttp) 这个实现。地址为： [
https://github.com/square/okhttp ](https://github.com/square/okhttp) 。

接着我们还得去看看volley的源码，由于我们的app兼容的最低版本是4.0，因此我们知道最终调用的是HurlStack：

    
    
        public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
    ...
            if (stack == null) {
                if (Build.VERSION.SDK_INT >= 9) {
                    stack = new HurlStack();
                } else {
                    // Prior to Gingerbread, HttpUrlConnection was unreliable.
                    // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                    stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
                }
            }
    ...
        }

因此我们只需要将HurlStack的相关代码修改即可，如下：

volley.java

    
    
    public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
    ...
            if (stack == null) {
                if (Build.VERSION.SDK_INT >= 9) {
                    // old way: stack = new HurlStack();
                		// http://square.github.io/okhttp/
                    stack = new HurlStack(null, null, new OkUrlFactory(new OkHttpClient()));
                } else {
                    // Prior to Gingerbread, HttpUrlConnection was unreliable.
                    // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                    stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
                }
            }
    ...
        }

HurlStack.java

    
    
    /**
     * An {@link HttpStack} based on {@link HttpURLConnection}.
     */
    public class HurlStack implements HttpStack {
    
        private final OkUrlFactory mOkUrlFactory; 
    
        /**
         * @param urlRewriter Rewriter to use for request URLs
         * @param sslSocketFactory SSL factory to use for HTTPS connections
         * @param okUrlFactory solution delete body(https://github.com/square/okhttp)
         */
        public HurlStack(UrlRewriter urlRewriter, SSLSocketFactory sslSocketFactory, OkUrlFactory okUrlFactory) {
            mUrlRewriter = urlRewriter;
            mSslSocketFactory = sslSocketFactory;
            mOkUrlFactory = okUrlFactory;
        }
        /**
         * Create an {@link HttpURLConnection} for the specified {@code url}.
         */
        protected HttpURLConnection createConnection(URL url) throws IOException {
    		if(null != mOkUrlFactory){
    			return mOkUrlFactory.open(url);
    		}
            return (HttpURLConnection) url.openConnection();
        }
    
    
        @SuppressWarnings("deprecation")
        /* package */ 
        static void setConnectionParametersForRequest(HttpURLConnection connection,
                Request<?> request) throws IOException, AuthFailureError {
            switch (request.getMethod()) {
                ...
                case Method.DELETE:
                    connection.setRequestMethod("DELETE");
                    addBodyIfExists(connection, request);
                    break;
                ...
                default:
                    throw new IllegalStateException("Unknown method type.");
            }
        }
    
    ...
    }

  

2015-04-26更新：

再次使用到需要使用到okhttp，回头看下上面的代码，不知道当时怎么想的，使用这么复杂的方法引入Okhttp，估计是脑袋进水了。再来看下这个方法：newRe
questQueue(Context context, HttpStack
stack)，有两个参数：context和HttpStack，这里是要传入自己的HttpStack就好了。那么我们用OKhttp的实现：

    
    
    /**
     * An {@link com.android.volley.toolbox.HttpStack HttpStack} implementation which
     * uses OkHttp as its transport.
     */
    public class OkHttpStack extends HurlStack {
      private final OkHttpClient client;
     
      public OkHttpStack() {
        this(new OkHttpClient());
      }
     
      public OkHttpStack(OkHttpClient client) {
        if (client == null) {
          throw new NullPointerException("Client must not be null.");
        }
        this.client = client;
      }
     
      @Override protected HttpURLConnection createConnection(URL url) throws IOException {
        return client.open(url);
      }   
    }

参考： [ https://gist.github.com/JakeWharton/5616899
](https://gist.github.com/JakeWharton/5616899)

##  2、关于（修改）volley的缓存

  

volley有完整的一套缓存机制。而目前我们想做个简单的需求：部分界面（几乎不会改动的）简单的做一定时间的缓存，研究了下代码发现很容易修改达到自己的目的（有
时间在分析下volley的缓存机制，这个一定要做）。简单来说修改一个地方：request.  parseNetworkResponse中的

HttpHeaderParser（此处突然感慨volley的设计TMD灵活了，想怎么改就怎么改）。  HttpHeaderParser修改后的代码如下：

    
    
    /**
     * 修改后的，用户处理缓存
     */
    public class BHHttpHeaderParser {
    
        /**
         * Extracts a {@link Cache.Entry} from a {@link NetworkResponse}.
         *
         * @param response The network response to parse headers from
         * @return a cache entry for the given response, or null if the response is not cacheable.
         */
        public static Cache.Entry parseCacheHeaders(NetworkResponse response, boolean isCustomCache) {
    ...
            if(isCustomCache){
            		softExpire = now + Config.HTTP_CACHE_TTL;
            } else {
    	        	if (hasCacheControl) {
    	            softExpire = now + maxAge * 1000;
    	        } else if (serverDate > 0 && serverExpires >= serverDate) {
    	            // Default semantic for Expire header in HTTP specification is softExpire.
    	            softExpire = now + (serverExpires - serverDate);
    	        }
            }
            
            Cache.Entry entry = new Cache.Entry();
            entry.data = response.data;
            entry.etag = serverEtag;
            entry.softTtl = softExpire;
            entry.ttl = entry.softTtl;
            entry.serverDate = serverDate;
            entry.responseHeaders = headers;
    
            return entry;
        }
    ...
    }

此处大家可以发现，我们主要是根据自定义的变量决定如何修改cache的TTL来达到自己的目的。

  

#  3、HttpUrlConnection与PATCH（2015-04-26）

  

在使用Volley发送PATCH请求的时候，我们可能会遇到这样的问题：Unknown method 'PATCH'; must be one of
[OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE]。这个时候你的第一反应是什么呢？是Volley不支持PATCH请
求吗？换成OkHttp是不是可以呢？查看了下Volley的源码，在HurlHttp.java中发现如下一段：

    
    
    /* package */ 
    static void setConnectionParametersForRequest(HttpURLConnection connection,
                Request<?> request) throws IOException, AuthFailureError {
            switch (request.getMethod()) {
                case Method.DEPRECATED_GET_OR_POST:
                    // This is the deprecated way that needs to be handled for backwards compatibility.
                    // If the request's post body is null, then the assumption is that the request is
                    // GET.  Otherwise, it is assumed that the request is a POST.
                    byte[] postBody = request.getPostBody();
                    if (postBody != null) {
                        // Prepare output. There is no need to set Content-Length explicitly,
                        // since this is handled by HttpURLConnection using the size of the prepared
                        // output stream.
                        connection.setDoOutput(true);
                        connection.setRequestMethod("POST");
                        connection.addRequestProperty(HEADER_CONTENT_TYPE,
                                request.getPostBodyContentType());
                        DataOutputStream out = new DataOutputStream(connection.getOutputStream());
                        out.write(postBody);
                        out.close();
                    }
                    break;
                case Method.GET:
                    // Not necessary to set the request method because connection defaults to GET but
                    // being explicit here.
                    connection.setRequestMethod("GET");
                    break;
                case Method.DELETE:
                    connection.setRequestMethod("DELETE");
                    break;
                case Method.POST:
                    connection.setRequestMethod("POST");
                    addBodyIfExists(connection, request);
                    break;
                case Method.PUT:
                    connection.setRequestMethod("PUT");
                    addBodyIfExists(connection, request);
                    break;
                case Method.HEAD:
                    connection.setRequestMethod("HEAD");
                    break;
                case Method.OPTIONS:
                    connection.setRequestMethod("OPTIONS");
                    break;
                case Method.TRACE:
                    connection.setRequestMethod("TRACE");
                    break;
                case Method.PATCH:
                    connection.setRequestMethod("PATCH");
                    addBodyIfExists(connection, request);
                    break;
                default:
                    throw new IllegalStateException("Unknown method type.");
            }
        }

通过这段代码，我们知道，Volley对PATCH还是支持的。在细看下错误这个是有HttpUrlConnection抛出的。因此我们需要在这方面下手。这里有一
个参考：

[ https://github.com/adriancole/retrofit/commit/e704b800878b2e37f5ac98b0139cb4
994618ace0 ](https://github.com/adriancole/retrofit/commit/e704b800878b2e37f5a
c98b0139cb4994618ace0)  

  

  

以后有其他关于volley的总结都记录在此。  

