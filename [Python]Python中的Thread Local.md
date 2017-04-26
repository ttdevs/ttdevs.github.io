日常开发中难免会用到多线程编程，这个时候我们可能会遇到线程同步的问题。当然，这不是我们今天要讲的～～

在线程内部，我们可能会用到线程内部的全局变量，这个也是比较常用的，因为可以全局访问。但是，如果线程内部的方法比较多，这个时候就需要我们一层层的传递，一层还好，二层三层甚至更多的时候就比较郁闷了。如果不传递，那么我们可能很难拿到线程内部的全局变量，这个时候我们就希望有个应用内的全局变量，供我们随时调用，thread local就可以满足我们的需求。看一个demo：

``` python
import threading

local = threading.local()


def end_learning():
    print('%s is finish.(Current Thread: %s)' % (local.name, threading.current_thread().name))


def start_learning():
    print('%s is learning.(Current Thread: %s)' % (local.name, threading.current_thread().name))
    end_learning()


def learn_python(name):
    local.name = name
    start_learning()


if __name__ == '__main__':
    thread1 = threading.Thread(target=learn_python, args=('ttdevs1',), name='Thread-A')
    thread1.start()

    thread2 = threading.Thread(target=learn_python, args=('ttdevs2',), name='Thread-B')
    thread2.start()
```

执行结果如下：
``` shell
ttdevs1 is learning.(Current Thread: Thread-A)
ttdevs1 is finish.(Current Thread: Thread-A)
ttdevs2 is learning.(Current Thread: Thread-B)
ttdevs2 is finish.(Current Thread: Thread-B)

Process finished with exit code 0
```

从上面的demo中，我们可以还可以看到：线程A和线程B之间的数据是没有相互干扰的。

看到这里你可能会感觉这个不是特别简单嘛。是的，这个就是很简单，通过查看源码我们可以发现，他的实现就是通过一个字典来存放我们不同线程的数据，我们也可以自己实现的。既然系统实现了，我们就可以直接拿来用了。

比如，在实际的应用中，我们可以为每个线程保存一个数据库连接，在线程的不同方法中使用就非常的方便了。

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

