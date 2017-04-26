
GPIO：General Purpose Input/Output pins on the Raspberry Pi

>General purpose input/output; in this specific case the pins on the Raspberry Pi and what you can do with them. So called because you can use them for all sorts of purposes; most can be used as either inputs or outputs, depending on your program.

## 0x00 GPIO

有些事情坚持了，就做出来了，不坚持，就放弃了，就这么简单。之前想写个树莓派的入门文章，一下子写了几篇之后，就停下来了。可能最近确是比较忙，或者，承认没有严格要求自己，好吧，拖了这么久，今天，再写一篇。

不管你信不信，树莓派最吸引我的就是它的GPIO口。即使是现在，想到GPIO还是非常的兴奋呢。为什么呢？对于通用计算机，我们能操作的基本都是USB、串口等一些接口。与这些接口相比，GPIO口有明显的不同，我们可以直接操作一个IO口的状态：是高电平还是低电平。根据这个特性，我们就可以完成首次接触单片机完成的小实验：控制我们的LED灯，制作跑马灯，模拟交通灯等等有趣的实验。如果你想通过其他设备来完成这些操作，可能需要掌握其他对初学者来说比较困难的技术。另外，大学时期有玩过 89C51，对IO口还是恋恋不舍，曾经想通过其产生PWM信号控制舵机，很遗憾没实现，现在正好可以完成这个心愿。

## 0x01 从一个LED说开去

这是一篇很基础的文章，所以会比较通俗。首先我们通过LED的Demo来了解下什么是GPIO。先看一张原理图：

![LED Demo](http://upload-images.jianshu.io/upload_images/1801981-1dd47f2ab18be25f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一个非常简单的电路：一个发光二极管、一个电阻（300Ω左右，忘记怎么算的了）、一个Rapsberry Pi。由于Rapsberry Pi GPIO口输出电压是3V3，所以我们串一个电阻来保护我们的LED。当我们通过代码控制链接电阻的那个GPIO口状态时，就可以使LED在亮和灭之间切换。

``` python3
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

led = 23
GPIO.setup(led, GPIO.OUT)

try:
    while(True):
        GPIO.output(led, 1)
        time.sleep(1)
        GPIO.output(led, 0)
        time.sleep(1)
except Exception as e:
    print(e)
finally:
    GPIO.cleanup()
```

运行上面的代码，我们就可以看到LED两秒一个循环的亮灭。哈哈，是不是特有成就感，传说中的点亮小灯泡终于实现了～～  然并卵，这并没有什么实际的用处。下面再介绍一个屌屌的demo，通过`HC-SR501`人体红外传感器检测控制LED的亮灭，实际的例子的就是很多自动门，站在门下面的时候门会自动打开。 `HC-SR501` 它是长这个样子滴：

![hcsr501 1](http://upload-images.jianshu.io/upload_images/1801981-b20c464659eac97e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![hcsr501 2](http://upload-images.jianshu.io/upload_images/1801981-4f71a6cfafddac73.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(PS:以上图片来自两个淘宝卖家)

三个针脚：VCC、OUT、GND，另外还有一个是否连续监测的针脚，连续时是H，不连续时是L，默认是H，看你买的具体设计了。另外还有两个调节旋钮，如上图。其实这个传感器特简单，供电电压5到20V，输出0或者3V3两个状态，不用与 Raspiberry Pi 连接，也可以直接当作开关使用。另外我还接了一个自锁开关，控制整个电路的开关。如下图：

![hcsr501 switcher](http://upload-images.jianshu.io/upload_images/1801981-f0bc7fe3d2041548.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接着上源码：

``` python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

led = 25
GPIO.setup(led, GPIO.OUT)
swtich = 22
GPIO.setup(swtich, GPIO.IN, GPIO.PUD_UP)
hcsr501 = 23
GPIO.setup(hcsr501, GPIO.IN, GPIO.PUD_UP)

try:
    GPIO.output(led, 0)
    while(True):
        print(str(GPIO.input(swtich)) + '||' + str(GPIO.input(hcsr501)))
        if GPIO.input(swtich) == True:
            if GPIO.input(hcsr501) == True:
                GPIO.output(led, 1)
            else:
                GPIO.output(led, 0)
        else:
            GPIO.output(led, 0)

        time.sleep(1)
except Exception as e:
    print(e)
finally:
    GPIO.cleanup()
``` 

当我们把开关打开，经过人体红外传感器的时候LED（这个地方我用的是3V3的高亮LED，所以没有接电阻）就会亮。把这个东西放在房间里，晚上下班开门就可以自动开灯了。如果你有兴趣，还可以把里面的开关换成一个光线传感器，那样就更完美了～～

运行上面的代码，我们需要安装 `RPi.GPIO` 库，Raspbian 默认是安装好了。简单的浏览上面的代码，我们可以发现，首先设置 GPIO mode，这里设置为BCM；然后定义GPIO的状态，GPIO口我们可以定义两种状态，GPIO.IN 和 GPIO.OUT，分别对应输入和输出；最后就是操作我们的GPIO口了。由于`RPi.GPIO` 库已经给我们封装复杂的操作，所以我们用起来特别简便。更多可 `RPi.GPIO` 信息可参考[这里](https://pypi.python.org/pypi/RPi.GPIO/0.6.2)。

## 0x02 Raspberry Pi的GPIO针脚

根据上面的demo程序，我们再来详细的介绍下GPIO。

- GPIO.setmode()

	指定针脚的编号方式，这里有两种选择：	
	1. GPIO.BOARD
	
		使用针脚的物理顺序来编号
	
	2. GPIO.BCM
		
		Broadcom SoC 的编码方式
	
	具体的可参考下面这张图，其中 `Pin`  列代表的是 `GPIO.BOARD` 方式的编码， 而 `NAME` 列代表的是`GPIO.BCM` 方式的编码。
	
        ![GPIO Pi3](http://upload-images.jianshu.io/upload_images/1801981-de0f86b2b6be0658.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)	

	>不同版本的 Raspberry Pi 会有不同的针脚分布，除查阅相关资料外，可以执行下面命令查看：
	> pi@raspberrypi:~ $ gpio readall
		
- GPIO.setwarnings(True or False)
	
	当我们有多个脚本在操作同一个GPIO口时可能会出现警告，通过这个设置来选择是否忽略
	
- GPIO.setup(channel, GPIO.OUT, initial=GPIO.HIGH)
	
	设置GPIO口的状态，参数分别为：针脚序号、输入还是输出、初始状态高电平还是低电平。
	
- GPIO.input(channel)

	获取指定GPIO口的状态
	
- GPIO.output(channel, GPIO.HIGH)

	操作GPIO口的输出状态

- GPIO.cleanup()

	清除当前操作的所有GPIO口的状态，恢复为输入状态。好的做法是每次使用完都清除一下。
	
这就是一个简单完整的使用 python 操作 GPIO的流程。虽然看到的现象，看到的代码，但是你还会问：GPIO到底是个啥样的东西，我们设置不同的状态时它的电压时怎样的？它的驱动能力如何？什么样的操作可能导致烧毁GPIO？

Raspberry Pi 2 Model B 使用了 BCM2836芯片，Raspberry Pi Model A, B, B+, Compute Module and Raspberry Pi Zero使用了BCM2835芯片，这里我们以BCM2835为例（作为一般使用者，可以不关心这个）。BCM2835有3个GPIO bank，这个3个bank都有自己的VDD，并且由3V3电压提供支持。`如果我们的输入电压超过3V3，就有可能烧毁SoC上的GPIO bank`。由于我也是业余的，说的再专业我也听不懂，因此简单的来说，使用外部传感器，它的输入信号电压是3V3的就没问题。当然，传感器的驱动电压可以不是3V3。比如上面使用的 `HC-SR501` 它的驱动电压是5V，输入信号电压是3V3，这样接在外面的 Raspberry Pi 就完全没问题，而且不需要额外电源为其供电，因为 Raspberry Pi 2 Model B 有两个5V的针脚（就是右上角的两个，2和4）。

更多的关于GPIO的上拉下拉悬浮等状态，可以参考[这里](http://dreamcolor.net/archives/rpi-gpio-module-inputs.html)的介绍。

## 0x03 GPIO进阶

对于控制简单的设备，比如LED，使用 RPi.GPIO 已经够用了。但是当我们进一步研究之后会发现RPi.GPIO并不是那么完美，在它的官方文档上有这么两段话：

>Note that this module is unsuitable for real-time or timing critical applications. This is because you can not predict when Python will be busy garbage collecting. It also runs under the Linux kernel which is not suitable for real time applications - it is multitasking O/S and another process may be given priority over the CPU, causing jitter in your program. If you are after true real-time performance and predictability, buy yourself an Arduino http://www.arduino.cc !
>Note that the current release does not support SPI, I2C, hardware PWM or serial functionality on the RPi yet. This is planned for the near future - watch this space! One-wire functionality is also planned.

从上面我们可以了解到 RPi.GPIO 并不适合实时性要求高的应用，也不适合周期计数的应用。python本身性能就不是非常理想，比如当我们尝试用python去操作DHT11（温湿度传感器）就非常的困难，因为很难和它的时序保持同步。而且linux并不是实时操作系统，运行在linux内核之上，无法避免被其他进程抢占CPU。目前版本的 RPi.GPIO 并不支持SPI、I2C、硬件PWM、串口等。如果你希望使用这些接口，那就需要考虑换一个库或者换一种语言了。具体的是用，在以后的文章中会做介绍。


## 0xFF 参考

1. https://github.com/raspberrypi/documentation
2. https://zh.scribd.com/doc/101830961/GPIO-Pads-Control2
3. https://pypi.python.org/pypi/RPi.GPIO
4. http://dreamcolor.net/archives/rpi-gpio-module-pwm.html

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


