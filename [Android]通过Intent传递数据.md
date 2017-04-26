
## 0x00 需求

日常开发中，我们经常遇到通过 Intent 来传递数据（如果你使用Bundle也是一样的），默认情况，我们可以看到支持的数据类型：
 
![intent](http://img.blog.csdn.net/20160807214929418)

- 基本类型

    boolean、byte、char 、short、double、float、int、long
    
- 字符串类型

    String、String[]、CharSequence、CharSequence[]

- 序列化类型

    Serializable、Parcelable
    
- 其他类型

    八种基本类型的数组、IntegerArray、StringArray、CharSequenceArray、ParcelableArray
    
对于基本类型我们可以不做任何处理即可在Activity或者Service中传递。那么对于自定义类型，我们怎么做呢？


## 0x01 何谓序列化

当我们需要对JVM中的java对象进行转储时，就需要将其转化成二进制序列，然后才能存储到外部。这个序列中保存了java对象的类型，数据，数据类型等。当需要的时候，再对存储的二进制序列进行反序列化，即可将其还原。常见的场景比如：存储JVM中对象，通过Socket传递java对象，RMI等等。


## 0x02 String类型

> 为什么把这个放在第一位呢？
原因很简单：简单

基本类型不用说，直接传递即可，对于我们的自定义类型，如果你跟我一样，不太喜欢（熟悉 T-T）Serializable和Parcelable，那用String类型最简单了。其实这个说法还有个前提，就是我们的项目中需要集成json工具（不过至少有gson）。使用方法如下：首先通过json工具将我们的Object转换成json字符串，然后在接收处在通过json工具还原为Object。

从效率来看，会比下面两种情况低。但是对初学者来说，会比下面两种用起来简单。目前还没有遇到出错的问题（可能时因为我传递的数据比较小比较简单吧）。


## 0x02 Serializable

Java中经常会遇到 Serializable，比如这样：

``` java
public class Person implements Serializable {
    public String name;
    public int age;
    public boolean login;
    public float score;
}
```

有些地方需要进行序列化时，IDE会提示我们实现一个接口：`Serializable`，然后我们乖乖地加上这个接口即可（有时候IDE还会自动生成一个静态常量：`private static final long serialVersionUID = 23333333333333333333L; `）。如果你没有去追究为什么，那么事情就此结束。通过Intent传递数据时，也是如此。好了，我们进入下一部分～～

哈哈，哪能这么简单。你可能会问，为什么加个这个接口即可？Serializable是一种 标记接口，当我们实现这个接口之后，系统会自动通过反射机制将我们对象的成员进行序列化。这里会涉及一些其他信息，比如：serialVersionUID、静态常量的序列化等等，更多信息请参考[这里](http://www.ibm.com/developerworks/cn/java/j-lo-serial/index.html)。


## 0x03 Parcelable

Parcelable是android特有的处理序列化的方法。先看简单的使用：

 - 创建一个类：
 
 	``` java
public class User implements Parcelable {
	    public String userName;
	    public int age;
	    public boolean login;
	    public float score;
}
 	```
 - 使用AndroidStudio的代码生成功能生成如下 模版 代码：
 
 	``` java
 public class User implements Parcelable {
    public String userName;
    public int age;
    public boolean login;
    public float score;

    protected User(Parcel in) {
        userName = in.readString();
        age = in.readInt();
        login = in.readByte() != 0;
        score = in.readFloat();
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(userName);
        dest.writeInt(age);
        dest.writeByte((byte) (login ? 1 : 0));
        dest.writeFloat(score);
    }

    @Override
    public int describeContents() {
        return 0;
    }

    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in) {
            return new User(in);
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };
}
 	```
 
 对比上面的两段代码，我们可以发现：
 
 - 构造方法中接收一个 Parcel 对象，通过这个对象对成员变量赋值
 - 实现了 `Parcelable` 接口的方法
 	
 	- 在 `writeToParcel` 方法中将我们的成员变量写入 `Parcel`
 	- 在 `describeContents` 中返回一个数字 0
 		
 		> Describe the kinds of special objects contained in this Parcelable instance's marshaled representation. For example, if the object will include a file descriptor in the output of writeToParcel(Parcel, int), the return value of this method must include the CONTENTS_FILE_DESCRIPTOR bit.
 	
 - 创建了一个 `public static final` 常量 `CREATOR`

这样我们就可以通过Intent传递对象了。

在android的很多文章中都可以看到，Intent中传递数据，强烈推荐使用  `Parcelable` 他的效率会比 `Serializable`更高（比如后者没有使用反射等等）。当然没有完美的事情，`Parcelable` 并不能完全取代`Serializable`，比如将JVM对象存储到磁盘等等。但是对于Android开发中的通过Intent传递数据来说，已经足够。


## 0xFF 参考

1. http://stackoverflow.com/questions/3611843/is-using-serializable-in-android-bad/3612364#3612364
2. http://www.ibm.com/developerworks/cn/java/j-lo-serial/index.html
3. https://developer.android.com/reference/android/os/Parcelable.html
4. http://blog.csdn.net/js931178805/article/details/8268144

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

