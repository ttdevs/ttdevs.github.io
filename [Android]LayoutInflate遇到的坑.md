
## 0x01 问题

``` java
    private void initVoiceItem() {
        viewMusicList.removeAllViews();
        
        int localResource = SysPreferences.getAlarmVoiceResource();
        LayoutInflater inflater = LayoutInflater.from(this);
        for (int i = 0; i < VOICE_KEY.length; i++) {
            View view = inflater.inflate(R.layout.item_voice_name, viewMusicList); // TODO: 2017/2/10
            view.setOnClickListener(mClickListener);
            ...
        }
    }

    private final View.OnClickListener mClickListener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            int size = viewMusicList.getChildCount();
            for (int i = 0; i < size; i++) {
                View view = viewMusicList.getChildAt(i);
                ...
                if (v == view) {
                    {...1...}
                } else {
                    {...2...}
                }
            }
        }
    };
```

有上面一串代码，你能发现有什么问题吗？

嗯嗯，是这样的：只会执行代码块{1}，并没有像我们期待的那样点击的时候执行到代码块{2}中去。没有细究，通过下面的代码直接跨过去：

``` java
private void initVoiceItem() {
   viewMusicList.removeAllViews();
   
   int localResource = SysPreferences.getAlarmVoiceResource();
   LayoutInflater inflater = LayoutInflater.from(this);
   for (int i = 0; i < VOICE_KEY.length; i++) {
       View view = inflater.inflate(R.layout.item_voice_name, null); // TODO: 2017/2/10
       view.setOnClickListener(mClickListener);
       ...
       viewMusicList.addView(view);
   }
}
```

虽然不知道什么原因，但是找到解决办法，不过还是挺惭愧的······


## 0x02 分析

很多人可能跟我一样，开始学的时候，记住该怎么写，这个方法是干嘛的。但是很难避免会记错某个方法，就像我们会写错某个字一样，当别人纠正的时候才知道，这个字自己已经写错十几活着几十年了。来看一下 `View.inflate()` 这个方法：

``` java
/**
* Inflate a new view hierarchy from the specified xml resource. Throws
* {@link InflateException} if there is an error.
* 
* @param resource ID for an XML layout resource to load (e.g.,
*        <code>R.layout.main_page</code>)
* @param root Optional view to be the parent of the generated hierarchy.
* @return The root View of the inflated hierarchy. If root was supplied,
*         this is the root View; otherwise it is the root of the inflated
*         XML file.
*/
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root) {
   return inflate(resource, root, root != null);
}
```

如果我们传入了 `root` View，那么返回的就是root View， 如果不传，则返回根据布局文件生成的 View。而根据我上面的代码，显然被我错误的理解，无论传不传 `root` View，返回的都是根据布局文件生成的 View，而我就这么相安无事的用了好几年······


## 0x03 你以为这就这样结束了

如果你执行我的错误代码，你会看到下面这个图：

![这里写图片描述](http://img.blog.csdn.net/20170212223305559?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这个这个图有两个信息：

- 后面几个Item显示的名称是错误的
- 点击某个Item，其他几个Item的checkbox也被选择

我们按照错误的代码执行的逻辑进行分析。首先：

``` java
for (int i = 0; i < VOICE_KEY.length; i++) {
  View view = inflater.inflate(R.layout.item_voice_name, viewMusicList);
  ...
}
```

这段代码会循环多次，将inflate后的布局添加到viewMusicList，这样viewMusicList下面就有多个 `item_voice_name` 因此最终给我们展现的结果就是看到有多个Item。

对于第一个问题，我们需要从 `view.findViewById()` 说起，从View中你会发现这两段代码：

``` java
/**
* Look for a child view with the given id.  If this view has the given
* id, return this view.
*
* @param id The id to search for.
* @return The view that has the given id in the hierarchy or null
*/
@Nullable
public final View findViewById(@IdRes int id) {
   if (id < 0) {
       return null;
   }
   return findViewTraversal(id);
}

/**
* {@hide}
* @param id the id of the view to be found
* @return the view of the specified id, null if cannot be found
*/
protected View findViewTraversal(@IdRes int id) {
   if (id == mID) {
       return this;
   }
   return null;
}
```

是不是太简单，根本没找到我们期待的逻辑——ViewGroup中怎么处理，细心的你会发现ViewGroup重写了 `findViewTraversal()` 方法：

``` java
@Override
protected View findViewTraversal(@IdRes int id) {
   if (id == mID) {
       return this;
   }

   final View[] where = mChildren;
   final int len = mChildrenCount;

   for (int i = 0; i < len; i++) {
       View v = where[i];

       if ((v.mPrivateFlags & PFLAG_IS_ROOT_NAMESPACE) == 0) {
           v = v.findViewById(id);

           if (v != null) {
               return v;
           }
       }
   }

   return null;
}
```

从这段代码可以看出，在ViewGroup中根据ID查找，找到就返回，而找到的永远是最前面的View。这就解释了为什么第一个Item和其他的不同了。

（对于点击某个Item，其他Item也出现波纹效果，猜测可能是因为波纹效果是根据ID来实现的。TODO ）


## 0x04 总结

上面遇到的View inflate是我个人遇到的问题，主要是因为对基础知识掌握有问题。另外在使用inflate的时候，可能还会遇到LayoutParam设置无效的问题，这个可以通过套一个View的方式解决，仅此记录。 

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


