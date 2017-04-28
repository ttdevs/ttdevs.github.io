
---
title: 「Android」各种小记录
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


## 0x00 小记录

这里是一些零碎知识点


## 0x01 sqlite中处理单引号

所有单引号换成双单引号，如：

`content.replace("'", "''");`

这样是不行的，临时抱佛脚，换成了带"？"的通配形式


## 0x02 SimpleCursorAdapter.notifyDataSetChanged无效

可以使用SimpleCursorAdapter的changeCursor方法：

- [http://stackoverflow.com/questions/1985955/android-simplecursoradapter-not-updating-with-database-changes](http://stackoverflow.com/questions/1985955/android-simplecursoradapter-not-updating-with-database-changes)
- [http://stackoverflow.com/questions/14034770/using-notifydatasetchanged-on-simplecursoradapter-does-not-work](http://stackoverflow.com/questions/14034770/using-notifydatasetchanged-on-simplecursoradapter-does-not-work)  


## 0x03 listview隐藏分割线

- `setDriver(null)` 
- 在xml文件中属性设置为 `@null`


## 0x04 sqlite中replace into无效

之前写好的代码调试都通过了，今天突然发现不能运行了：

>sqlite，表中创建唯一索引，然后使用replace into实现插入或者更新，结果变成了只插入

折腾了几个小时，最后改掉了表中一个名为id的字段，实在是无语


## 0x05 临时写java的Thread封装

``` java
public abstract class BaseThread extends Thread {

    private volatile boolean mQuit = false;

    public BaseThread() {

    }

    /**
     * prepare if needed
     */
    public boolean prepare() {

        return true;
    }

    /**
     * run body
     */
    public abstract void execute() throws Exception; // third modify,second modify maybe not neccessary

    /**
     * recycle if needed
     */
    public void recycle() {

    }

    /**
     * stop thread
     */
    public final void quit() {
        mQuit = true;
        interrupt();
    }

    @Override
    public void run() {
        // TODO
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        if (!prepare()) {
            return;
        }
        while (true) {
            if (mQuit) { // second modify
                recycle();
                return;
            }
            try {
                execute();
            } catch (Exception e) {// } catch (InterruptedException e) {
                if (mQuit) {
                    recycle();
                    return;
                }
                continue;
            }
        }
    }
}
```  


## 0x06 Volley中设置连接超时

设置超时的地方：

HurlStack：

``` java
/**
* Opens an {@link HttpURLConnection} with parameters.
*
* @param url
* @return an open connection
* @throws IOException
*/
private HttpURLConnection openConnection(URL url, Request<?> request) throws IOException {
   HttpURLConnection connection = createConnection(url);

   int timeoutMs = request.getTimeoutMs();
   connection.setConnectTimeout(timeoutMs);
   connection.setReadTimeout(timeoutMs);
   connection.setUseCaches(false);
   connection.setDoInput(true);

   // use caller-provided custom SslSocketFactory, if any, for HTTPS
   if ("https".equals(url.getProtocol()) && mSslSocketFactory != null) {
       ((HttpsURLConnection) connection).setSSLSocketFactory(mSslSocketFactory);
   }

   return connection;
}
```    

HttpClientStack：  

``` java
@Override
public HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders)
        throws IOException, AuthFailureError {
    HttpUriRequest httpRequest = createHttpRequest(request, additionalHeaders);
    addHeaders(httpRequest, additionalHeaders);
    addHeaders(httpRequest, request.getHeaders());
    onPrepareRequest(httpRequest);
    HttpParams httpParams = httpRequest.getParams();
    int timeoutMs = request.getTimeoutMs();
    // TODO: Reevaluate this connection timeout based on more wide-scale
    // data collection and possibly different for wifi vs. 3G.
    HttpConnectionParams.setConnectionTimeout(httpParams, 5000);
    HttpConnectionParams.setSoTimeout(httpParams, timeoutMs);
    return mClient.execute(httpRequest);
}
```

Request中：

``` java
/**
* Sets the retry policy for this request.
*/
public void setRetryPolicy(RetryPolicy retryPolicy) {
   mRetryPolicy = retryPolicy;
}
```

RetryPolicy ：

``` java
/**
* Retry policy for a request.
*/
public interface RetryPolicy {

   /**
    * Returns the current timeout (used for logging).
    */
   public int getCurrentTimeout();

   /**
    * Returns the current retry count (used for logging).
    */
   public int getCurrentRetryCount();

   /**
    * Prepares for the next retry by applying a backoff to the timeout.
    *
    * @param error The error code of the last attempt.
    * @throws VolleyError In the event that the retry could not be performed (for example if we
    *                     ran out of attempts), the passed in error is thrown.
    */
   public void retry(VolleyError error) throws VolleyError;
}
```


## 0x07 Android与图片相关

[http://developer.android.com/training/displaying-bitmaps/index.html](http://developer.android.com/training/displaying-bitmaps/index.html)  


## 0x08 将eclipse项目中的文件夹移除SVN的控制（添加到svn:ignore）

以删除android项目中的bin文件夹为例

a、关闭项目的自动build：project-Build Automatically
b、删除bin文件夹
c、提交
d、打开 项目的自动build，这个时候会生成bin文件夹
e、将bin添加到svn:ignore：右键-Team-添加至svn:ignore
f、提交


## 0x09 屏幕旋转，Activity的成员变量被全部释放？！

貌似是这样子的


## 0x10 ViewPager频繁被回收

[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html#setOffscreenPageLimit\(int\))  

``` java
/**
* Set the number of pages that should be retained to either side of the
* current page in the view hierarchy in an idle state. Pages beyond this
* limit will be recreated from the adapter when needed.
*
* <p>This is offered as an optimization. If you know in advance the number
* of pages you will need to support or have lazy-loading mechanisms in place
* on your pages, tweaking this setting can have benefits in perceived smoothness
* of paging animations and interaction. If you have a small number of pages (3-4)
* that you can keep active all at once, less time will be spent in layout for
* newly created view subtrees as the user pages back and forth.</p>
*
* <p>You should keep this limit low, especially if your pages have complex layouts.
* This setting defaults to 1.</p>
*
* @param limit How many pages will be kept offscreen in an idle state.
*/
public void setOffscreenPageLimit(int limit) {
   if (limit < DEFAULT_OFFSCREEN_PAGES) {
       Log.w(TAG, "Requested offscreen page limit " + limit + " too small; defaulting to "
               + DEFAULT_OFFSCREEN_PAGES);
       limit = DEFAULT_OFFSCREEN_PAGES;
   }
   if (limit != mOffscreenPageLimit) {
       mOffscreenPageLimit = limit;
       populate();
   }
}
```


## 0x11 Android SQLite 分页查询，参数为页大小和起始编号

先看数据：

![](http://img.blog.csdn.net/20131126171733656?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

查询第0条之后的5条：
`select * from china_city where province = '北京' order by code limit 5 offset 0`

![](http://img.blog.csdn.net/20131126171744390?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

查询第5条之后的5条：
`select * from china_city where province = '北京' order by code limit 5 offset 5`

![](http://img.blog.csdn.net/20131126171750843?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  


## 0x12 获取进程内存 和 已使用内存

- 进程内存

``` java
int memClass = ((ActivityManager)context.getSystemService(Context.ACTIVITY_SERVICE)).getMemoryClass();

int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
```

- 已使用内存

    `Log.i("tag", where+"usedMemory: "+Debug.getNativeHeapSize()/ 1048576L);`

## 0x13 与联系人相关

[http://developer.android.com/training/contacts-provider/display-contact-badge.html ](http://developer.android.com/training/contacts-provider/display-contact-badge.html)  


## 0x14 设置字体大小

可以用setTextSize()的另外一种形式，可以指定单位：  
`setTextSize(int unit, int size)`

- TypedValue.COMPLEX_UNIT_PX: Pixels
- TypedValue.COMPLEX_UNIT_SP: Scaled Pixels
- TypedValue.COMPLEX_UNIT_DIP: Device Independent Pixels


## 0x15 交换两个变量的值

``` java    
date3 = date1;
date1 = date2;
date2 = date3;
```

``` java
if (date2 > date1) {
    date2 = date2 + date1;
    date1 = date2 - date1;
    date2 = date2 - date1;
}
```

``` java
date2 = date1 ^ date2;
date1 = date1 ^ date2;
date2 = date1 ^ date2;
```


## 0x16 获取URI真实的地址

``` java
private String getRealPath(Uri uri) {
   String path = "";
   String[] proj = {MediaStore.Images.Media.DATA};
   Cursor cursor = getActivity().getContentResolver().query(uri, proj, null, null, null);
   if (cursor != null && cursor.moveToFirst()) {
       path = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA));
   }
   return path;
}
```  


## 0x17 自定义Gridview，解决ScrollView中嵌套Gridview显示不正常的问题

网上到处都可以找到下面的解决办法，但是这样会有一个问题：当图片非常多的时候是一次加载完毕的，而我们希望的应该是本来的GridView可以通过滑动判断当前显示
条目的情况，在此记录一下。

``` java
/**
* 自定义Gridview，解决ScrollView中嵌套Gridview显示不正常的问题<br>
* 但是，这又产生了一个新的问题：所有的item都是一次加载完毕的，再想滑动加载就不行了
*
* @author ttdevs
*/
public class ImageGridView extends GridView {
   public ImageGridView(Context context) {
       this(context, null);
   }

   public ImageGridView(Context context, AttributeSet attrs) {
       this(context, attrs, 0);
   }

   public ImageGridView(Context context, AttributeSet attrs, int defStyle) {
       super(context, attrs, defStyle);
   }

   @Override
   public void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
       //super.onMeasure(widthMeasureSpec, heightMeasureSpec);
       int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
       super.onMeasure(widthMeasureSpec, expandSpec);
   }
}
```


## 0x18 .9图片的制作

一个图片只想横向拉伸。所以就想当然的只在左边画了一条横线，运行的时候怎么都是有黑线，没拉伸。

`最后解决：给左边也加一条竖线！`


## 0x20 listview点击背景不变

这个应该还是焦点的问题，在item的view最上层布局中添加一个属性：

`android:descendantFocusability="blocksDescendants"`


## 0x21 Volley Cookies

[http://stackoverflow.com/questions/16680701/using-cookies-with-android-volley-library](http://stackoverflow.com/questions/16680701/using-cookies-with-android-volley-library)


## 0x22 EditText设置readOnly

``` java
et.setCursorVisible(false); 
et.setFocusable(false); 
et.setFocusableInTouchMode(false);
```


## 0x23 保存对象到SharedPreferences

想保存一个List<JavaBean> 到SharedPreferences。网上找了一圈有一种说法利用set，或者base64，都感觉不好。朋友一句提示：json。然后豁然开朗。现转化成json，然后保存，效果不错。


## 0x24 获取字符

``` java
public String getCharacter(String str) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < str.length(); i++) {
        if (String.valueOf(str.charAt(i)).getBytes().length > 1) {
            sb.append(str.charAt(i));
        }
    }
    return sb.toString();
}
```


## 0x25 Android 4 按钮样式

`style="?android:attr/buttonBarButtonStyle"`


## 0x26 几个sqlite的sql语句

- select count(*) from notes
- create table notes_new as select * from notes;
- select count(*) from notes_new
- delete from notes_new where word like '% %';
- delete from notes_new where word like '%-%';
- delete from notes_new where phonetic like '';
- create table notes_new_new as select * from notes_new;
- select count(*) from notes_new_new
- delete from notes_new_new where _id not in (select min(_id) from notes_new_new group by word)
- select * from notes_new_new
- insert into notes_word(word,detail,created,phonetic) select word,detail,created,phonetic from notes_new_new;
- Create Table:

    ``` sql 
    CREATE TABLE notes_word(
            _id INTEGER PRIMARY KEY AUTOINCREMENT,
            word TEXT,
            detail TEXT,
            created INT,
            phonetic TEXT
    );
    ```

## 0x27 加密相关

- http://blog.csdn.net/androidsecurity/article/details/8666954  
- http://blog.csdn.net/baolong47/article/details/17335227  
- http://blog.csdn.net/guolin_blog/article/details/11952409  


## 0x28 FaceBook开源项目

[](http://facebook.github.io/rebound/)


## 0x29 ScrollView与高度固定的GridView或者ListView嵌套时，界面不显示在最上面

- gvGroup.setFocusable(false);

    lvGroup.setFocusable(false);在设置Adapter之前让其失去焦点

- svMain.fullScroll(ScrollView.FOCUS_UP);
    
    在合适的时候(如delay10ms)让ScrollView滚动到最顶端


## 0x30 获取view的高度

``` java
Rect rect = new Rect();
getDrawingRect(rect);
int height = rect.bottom - rect.top;
```

## 0x31 http://hold-on.iteye.com/blog/1943437

## 0x32 调用系统的浏览器打开网页

`区分大小写，http必须为小写`

``` java
String url = "http://www.baidu.com";
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse(url));
startActivity(intent);
```

## 0x33 HTTP POST 乱码

``` java
List<NameValuePair> postParameters = new ArrayList<NameValuePair>();
postParameters.add(new BasicNameValuePair("v", param));
postParameters.add(new BasicNameValuePair("s", Utils.MD5Encode(param, "")));
postParameters.add(new BasicNameValuePair("e", String.valueOf(encryptType)));

UrlEncodedFormEntity formEntity = new UrlEncodedFormEntity(postParameters, "UTF-8");
mPost.setEntity(formEntity);
```

## 0x34 读取联系人

``` java
/**
 * 读取本地通讯录（手机）<br>
 * 用户通讯录： 手机号码、联系人命名、备注信息、分组信息<br>
 * 耗时
 *
 * @param context
 * @return
 */
@SuppressLint("UseSparseArrays")
public static Map<String, ContactInfo> getContracts(Context context) {
    Map<String, ContactInfo> contactMap = new HashMap<String, ContactInfo>();
    try {

        ContentResolver resolver = context.getContentResolver();
        if (null == resolver) {
            return contactMap;
        }
        // 手机：姓名和手机号
        Cursor curosr = resolver.query(Phone.CONTENT_URI, new String[]{Phone.CONTACT_ID, Phone.DISPLAY_NAME, Phone.NUMBER}, null, null, null);
        if (null != curosr && curosr.moveToFirst()) {
            int keyId = curosr.getColumnIndex(Phone.CONTACT_ID);
            int keyName = curosr.getColumnIndex(Phone.DISPLAY_NAME);
            int keyNumber = curosr.getColumnIndex(Phone.NUMBER);
            do {
                String id = curosr.getString(keyId);
                String name = curosr.getString(keyName);
                String number = curosr.getString(keyNumber);
                ContactInfo info = new ContactInfo();
                info.setName(name);
                if (!TextUtils.isEmpty(number)) {
                    number = number.replace("-", "");
                    number = number.replace(" ", "");
                    info.setNumber(number);
                }
                contactMap.put(id, info);
            } while (curosr.moveToNext());
        }
        curosr.close();

        // SIM卡：姓名和手机号
        curosr = resolver.query(Uri.parse("content://icc/adn"), new String[]{Phone.CONTACT_ID, Phone.DISPLAY_NAME, Phone.NUMBER}, null, null, null);
        if (null != curosr && curosr.moveToFirst()) {
            int keyId = curosr.getColumnIndex(Phone.CONTACT_ID);
            int keyName = curosr.getColumnIndex(Phone.DISPLAY_NAME);
            int keyNumber = curosr.getColumnIndex(Phone.NUMBER);
            do {
                String id = curosr.getString(keyId);
                String name = curosr.getString(keyName);
                String number = curosr.getString(keyNumber);
                ContactInfo info = new ContactInfo();
                info.setName(name);
                info.setNumber(number);
                contactMap.put(id, info);
            } while (curosr.moveToNext());
        }
        curosr.close();

        // 备注
        curosr = resolver.query(Data.CONTENT_URI, new String[]{Data.CONTACT_ID, Data.DATA1}, null, null, null);
        if (null != curosr && curosr.moveToFirst()) {
            int keyId = curosr.getColumnIndex(Data.CONTACT_ID);
            int keyRemark = curosr.getColumnIndex(Data.DATA1);
            do {
                String id = curosr.getString(keyId);
                String remark = curosr.getString(keyRemark);
                ContactInfo info = contactMap.get(id);
                if (null != info) {
                    info.setRemark(remark);
                }
            } while (curosr.moveToNext());
        }
        curosr.close();

        // 获取所有分组
        Map<String, String> groupData = new HashMap<String, String>();
        curosr = resolver.query(Groups.CONTENT_URI, new String[]{Groups._ID, Groups.TITLE}, null, null, null);
        if (null != curosr && curosr.moveToFirst()) {
            int keyId = curosr.getColumnIndex(Groups._ID);
            int keyGroupName = curosr.getColumnIndex(Groups.TITLE);
            do {
                String id = curosr.getString(keyId);
                String name = curosr.getString(keyGroupName);
                groupData.put(id, name);
            } while (curosr.moveToNext());
        }
        curosr.close();

        // 关联联系人和分组
        curosr = resolver.query(Data.CONTENT_URI, new String[]{GroupMembership.CONTACT_ID, GroupMembership.GROUP_ROW_ID}, Data.MIMETYPE + " = ?",
                new String[]{GroupMembership.CONTENT_ITEM_TYPE}, null);
        if (null != curosr && curosr.moveToFirst()) {
            int keyId = curosr.getColumnIndex(GroupMembership.CONTACT_ID);
            int keyGroupId = curosr.getColumnIndex(GroupMembership.GROUP_ROW_ID);
            do {
                String id = curosr.getString(keyId);
                String groupId = curosr.getString(keyGroupId);
                String groupName = groupData.get(groupId);
                ContactInfo info = contactMap.get(id);
                if (null != info && !TextUtils.isEmpty(groupName)) {
                    info.setGroup(groupName);
                }
            } while (curosr.moveToNext());
        }
        curosr.close();

        Iterator<String> iterator = contactMap.keySet().iterator();
        while (iterator.hasNext()) {
            String id = iterator.next();
            ContactInfo info = contactMap.get(id);
            System.out.println(String.format("%s, %s, %s, %s.", info.getName(), info.getNumber(), info.getGroup(), info.getRemark()));
        }

        groupData.clear();
        groupData = null;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return contactMap;
}
```


## 0x35 单元测试——测试异步任务

``` java
public void testAsynchronous() {
    try {
        CountDownLatch signal = new CountDownLatch(1);
        new Thread(new Runnable() {

            @Override
            public void run() {
                Looper.prepare();
                signal.countDown();
                Looper.loop();
            }
        }).start();
        signal.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

## 0x36 GridLayout合并

``` xml
<GridLayout
    android:id="@+id/glNumber"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_below="@+id/etPassword"
    android:layout_centerHorizontal="true"
    android:alignmentMode="alignBounds"
    android:columnCount="3"
    android:orientation="horizontal"
    android:rowCount="4">

    <com.ttdevs.blur.CircleButton
        android:id="@+id/cbt1"
        android:layout_width="64dp"
        android:layout_height="64dp" />

    <com.ttdevs.blur.CircleButton
        android:id="@+id/cbt2"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_margin="10dp" />

    <com.ttdevs.blur.CircleButton
        android:id="@+id/cbt3"
        android:layout_width="64dp"
        android:layout_height="64dp" />

    <com.ttdevs.blur.CircleButton
        android:id="@+id/cbt0"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_columnSpan="2"
        android:layout_gravity="right" />

    <com.ttdevs.blur.CircleButton
        android:id="@+id/cbtDel"
        android:layout_width="64dp"
        android:layout_height="64dp" />
</GridLayout>
```


## 0x37 PopupWindow显示的时候左右不靠边（有边缝）

`setBackgroundDrawable(new ColorDrawable(0xa0000000));`


## 0x38 EditText限制可输入小数

`etInput.setInputType(InputType.TYPE_CLASS_NUMBER | InputType.TYPE_NUMBER_FLAG_DECIMAL);`

[](http://stackoverflow.com/questions/4276068/how-to-make-edittext-field-for-decimals)


## 0x39 隐藏键盘

隐藏键盘貌似在onDestory中调用无效，最后自己把它写在了onStop中

``` java
/**
 * 关闭键盘
 * @param context
 * @param windowToken
 * @param flag
 */
public static void close(Context context, IBinder windowToken, int flag) {
    InputMethodManager imm = (InputMethodManager) context
            .getSystemService(Context.INPUT_METHOD_SERVICE);
    imm.hideSoftInputFromWindow(windowToken, flag);
}
```


## 0x40 Android TextView 显示不全的问题

代码如下，当给TextView的属性设置成match_parent的时候，text中设置的文字比较长的时候会出现显示不全的问题

``` xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="top"
    android:orientation="horizontal"
    android:padding="@dimen/global_padding_normal">

    <ImageView
        android:id="@+id/iv_health_light"
        android:layout_width="@dimen/detail_light_width"
        android:layout_height="@dimen/detail_light_height"
        android:gravity="center_horizontal" />

    <TextView
        android:id="@+id/tv_appraise"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="@dimen/global_margin_normal"
        android:gravity="left|top"
        android:lineSpacingExtra="@dimen/global_line_space"
        android:textColor="@color/global_gray_dark"
        android:textSize="@dimen/global_font_small" />

</LinearLayout>
```


## 0x41 查看系统alarm(闹钟)队列

- Alarm事件队列

    `adb shell dumpsys alarm`

- 查看intent

    `adb shell dumpsys activity intents`

[](https://plus.google.com/+RomanNurik/posts/5w3Mi9EoniT)


## 0x42 singleTask与onNewIntent

singleTask与onNewIntent同时存在时，onNewIntent不是总被调用。比如：当前App启动过，第三方App再次启动时是不会调用onNewIntent的，再或者我们从任务列表中再次启动当前App也是不会触发onNewIntent的。  

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

