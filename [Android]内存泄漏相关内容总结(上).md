
## 0x00 忏悔

好久没有认真的写博客了，草稿箱中静静躺着好几篇，但总是不能把他们写完，一直没有写的心情把，罪过...

废话一句说完，进入今天的正题：内存泄露的相关知识。


## 0x01 内存问题

很多时候我们是需要查看自己的应用内存占用情况，比如当出现闪退，异常退出时：

- 界面内存占用：比如我们的启动画面，如果出现OOM的问题，这个时候可以通过差看内存情况，如果确是内存占用比较大就可以考虑做相应优化

- 内存泄漏的初步查找：这个和直接的OOM有些差别，虽然他最终也会导致OOM。方法：反复打开、关闭可能内存泄漏的界面，如果每次打开关闭之后内存都有所增加，就可以初步判断为内存泄漏。正常情况应用是进入界面内存占用有所增加，但是离开后内存会降到之前的水平。


## 0x02 内存泄漏与内存溢出(OOM)的区别与联系

内存溢出做android开发的各位应该是比较熟悉，图片加载稍微不小心就会导致out of memory的错误，这个错误出现的原因就是我们的应用去申请内存时系统无法再分配导致的。所以我们需要有节制的使用内存。

和C++相比，java的一个很大特点就是有GC，我们不需要去专注内存的释放与回收。但是，这也不是绝对的：当我们的代码出现问题导致GC无法及时的回收相关的内存时就会导致内存泄漏。这句不算太规范的描述（语文太差，见谅）他的重点是什么呢——GC无法及时的回收相关内存——这将导致我们的内存占用越来越多——最终的体现就是OOM。

希望这个简单的描述，各位可以理解什么是memory leak和OOM。


## 0x03 DDMS 给我们的提示

- logcat中的信息

关于内存的信息，logcat会给我们一些提示信息，比如：

``` shell
D/dalvikvm( 9050): GC_CONCURRENT freed 2049K, 65% free 3571K/9991K, external 4703K/5261K, paused 2ms+2ms
```

这行信息怎么理解？它包含哪些信息呢？通过 [DDMS的文档](https://developer.android.com/tools/debugging/debugging-memory.html#LogMessages) 我们可以得知这个信息的格式如下：

``` shell
D/dalvikvm(dvm pid): <GC_Reason> <Amount_freed>, <Heap_stats>, <External_memory_stats>, <Pause_time>
```

这行信息我们主要关注的是：GC_Reason，他主要有下面几种：

    GC_CONCURRENT()
    
    A concurrent garbage collection that frees up memory as your heap begins to
    fill up.
    
    GC_FOR_MALLOC(当我们的应用申请内存时触发)
    
    A garbage collection caused because your app attempted to allocate memory when
    your heap was already full, so the system had to stop your app and reclaim
    memory.
    
    GC_HPROF_DUMP_HEAP()
    
    A garbage collection that occurs when you create an HPROF file to analyze your
    heap.
    
    GC_EXPLICIT(当我们手动点击DDMS>heap>Cause GC按钮时会触发)
    
    An explicit garbage collection, such as when you call gc() (which you should
    avoid calling and instead trust the garbage collector to run when needed).
    
    GC_EXTERNAL_ALLOC()
    
    This happens only on API level 10 and lower (newer versions allocate everything in the Dalvik heap). A garbage collection for externally allocated memory (such as the pixel data stored in native memory or NIO byte buffers).

当我们在app在出现问题时可以根据这些信息做初步判断。比如频繁出现GC_FOR_AMLLOC则预示着我们一直在申请内存。  
  
- heap页签的内存信息

通过这个工具这个我们可以更详细的看到内存的分配与使用。启动方法：

`点击DDMS中我们的进程` > `上部的Update heap按钮` > `Cause GC` 即可，如下图：

![](http://img.blog.csdn.net/20141214202523491?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## 0x04 通过内存快照更精准的查找内存信息

当我们出现内存泄漏问题是，初学者可能会一头雾水，不知道如何解决，从何下手。这个时候我们就可以考虑使用mat对我们的内存进行详细的分析，找到内存泄漏点，做相应的处理。

- `Shallow size` 和 `Retained Size`

在进行内存分析之前我们需要首先了解这两个概念。

- Shallow size

Shallow size就是对象本身占用内存的大小，不包含其引用的对象。常规对象（非数组）的Shallow size有其成员变量的数量和类型决定。数组的shallow size有数组元素的类型（对象类型、基本类型）和数组长度决定。Shallow size of a set of objects represents the sum of shallow sizes of all objects in the set.在32位系统上，对象头占用8字节，int占用4字节，不管成员变量（对象或数组）是否引用了其他对象（实例）或者赋值为null它始终占用4字节。故此，对于String对象实例来说，它有三个int成员（3*4=12字节）、一个char[]成员（1*4=4字节）以及一个对象头（8字节），总共3*4+1*4+8=24字节。根据这一原则，对String a=”rosen jiang”来说，实例a的shallow size也是24字节。

- Retained Size

Retained Size=当前对象大小+当前对象可直接或间接引用到的对象的大小总和(间接引用的含义：A->B->C, C就是间接引用)。换句话说，Retained Size 就是当前对象被GC后，从Heap上总共能释放掉的内存。
不过，释放的时候还要排除被GC Roots直接或间接引用的对象。他们暂时不会被被当做Garbage。为了更好的理解retained size，不妨看个例子。把内存中的对象看成下图中的节点，并且对象和对象之间互相引用。这里有一个特殊的节点GC Roots，这就是reference chain的起点。

![](http://img.blog.csdn.net/20141214203439460?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
![](http://img.blog.csdn.net/20141214203446937?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

从obj1入手，上图中蓝色节点代表仅仅只有通过obj1才能直接或间接访问的对象。因为可以通过GC Roots访问，所以左图的obj3不是蓝色节点；而在右图却是蓝色，因为它已经被包含在retained集合内。

所以对于左图，obj1的retained size是obj1、obj2、obj4的shallow size总和；右图的retained size是obj1、obj2、obj3、obj4的shallow size总和。

对于obj2，它的retained size是：在左图中，是obj2和obj4的shallow size的和；在右图中，是obj2、obj3和obj4的shallow size的和。

总之，retained size是一个整体度量，有助于理解内存结构和对象图中的依赖关系并找到根节点。

- HPROF file——内存快照文件

首先这个文件可以通过点击 DDMS 中 Update heap 右侧的 Dump HPROF file 得到（这个有时候可能会比较慢，特别是内存占用高的时候，需要等一会，出现hprof: dumping heap strings to "[DDMS]".即代表已经触发）。

得到这个文件之后，我们还需要使用SDK的platform-tools文件夹下的“hprof-conv”工具进行转换，然后就可以使用内存分析工具查看了。转换方法：

`hprof-conv heap-original.hprof heap-converted.hprof`


## 0x05 内存分析工具MAT

这个是专门做内存分析的工具。通过它我们可以打开刚才转换后的到hprof文件，进行内存分析。每次都点击保存，转换，然后打开还是比较麻烦的。这个时候我们可以考虑
使用eclipse的mat插件。

这篇就先写到这里。下一篇简单介绍如何通过MAT查找内存泄漏的问题。  
  

## 0xFF 参考

1. [https://developer.android.com/tools/debugging/debugging-memory.html](https://developer.android.com/tools/debugging/debugging-memory.html)
2. [http://blog.csdn.net/kingzone_2008/article/details/9083327](http://blog.csdn.net/kingzone_2008/article/details/9083327)
3. [http://help.eclipse.org/luna/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Fconcepts%2Fshallowretainedheap.html&resultof=%22Shallow%22%20%22shallow%22%20%22size%22%20](http://help.eclipse.org/luna/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Fconcepts%2Fshallowretainedheap.html&resultof=%22Shallow%22%20%22shallow%22%20%22size%22%20)

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

