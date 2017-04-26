
## 0x00 为什么要装(Bi)

装逼遭雷劈。

上一篇介绍了重装macOS的方法。为什么会有上一篇？说到底就一个字：跑 python web 应用出问题了。把macOS从旧版本升级到新版本 10.12，之前跑的好好的web应用现在各种问题，所以，重装吧。然并卵，重装后问题依旧 😢，比如：

`ImportError: No module named '_sqlite3'`
`ImportError: No module named 'pysqlite2'`

等等一堆问题。好吧，废话说多了，这里介绍下该怎么办。

## 0x01 安装

情况是这样的，全新安装macOS，同时升级 xcode 到最新版8 (8A218a)，使用 homebrew 安装 python3 ：

`brew install python3`

装好之后还是各种model找不到，缺少很多东西，尝试了各种方法，依然无法解决。

在反复重装 python3 的时候，看到这样一个提示：`xcode-select --install`。虽然不知道这个鬼东西是干嘛，但是确定一点，之前我有安装过。所以，就在命令行执行了一下，然后再重新安装 python3，他 就，就，就 可以了……

## 0x02 Command Line Tools

`xcode-select --install` 这条命令是的作用是安装 `Command Line Tools`。找了半天，没看到哪有详细的介绍，在[这里][tools]看到一个简短的介绍：

> The Command Line Tools Package is a small self-contained package available for download separately from Xcode and that allows you to do command line development in OS X. It consists of two components: OS X SDK and command-line tools such as Clang, which are installed in /usr/bin.

简单来说，应该就是一套工具集合。他会被安装在下面两个位置：

`/Library/Developer/CommandLineTools/usr/bin`
`/usr/bin`

在这里你会看到一堆常见的命令。看到这些之后，恍然大悟，homebrew好像是下的python3的源码，然后本地编译安装的。如果缺少了这些命令，可能就会出错。

好了，终于可以愉快的使用 python 了。

------
[tools]: https://developer.apple.com/library/prerelease/content/technotes/tn2339/_index.html#//apple_ref/doc/uid/DTS40014588-CH1-WHAT_IS_THE_COMMAND_LINE_TOOLS_PACKAGE_

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

