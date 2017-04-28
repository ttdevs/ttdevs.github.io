
---
title: 「macOS」macOS安装
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - macOS
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x00 要不要重装？

对于重装系统，我是拒绝的。

想起刚买Mac那会，总是用Window的思维来理解Mac，费了老大的劲来对磁盘进行分区，最后分了两个区：用户目录放在一个区，其他文件放在另一区。感觉自己好屌，但是没过多久发现下面两个问题：

- 重装系统，之前用户目录的文件读取权限可能会有问题
	
	别问我为什么，知道重装可以解决大部分问题就好了
	
- 随着时间的流逝，开始预留给 系统文件和App所在分区 越来越不够用

最后总结：Mac 很少坏，所以，分区并没什么卵用，只会给您的使用带来麻烦。

好了，言归正传，自从升级了macOS 就出现各种小问题，python用不了了，adb经常断，系统变卡了，每个小问题都要去整一下。想想还是算了，花点时间，在另一台 Mac Mini 上重新安装，然后用 Time Machine 还原到 Mac Book上。


## 0x01 系统安装

电脑上有一份最新的macOS安装文件，打算去做安装U盘。下班回去准备操作的时候，发现做U盘安装盘的速度好慢慢好慢啊。想，还是算了吧，睡觉之前使用网络安装，第二天醒来应该可以装好。

这里附一个 macOS 安装文件下载地址：[Install macOS Sierra.app][macOS]。

### 网络安装
	
开机按住 Option(Alt) 键，进入启动引导，选择那个虚拟盘（可能需要输入WIFI密码，我用的网线），先将主硬盘格式化（这将丢失所有数据，请提前做好备份），然后选择第二项安装系统，按照提示点击各种 继续、同意 之类，就可以去睡觉了。


![Setup](http://upload-images.jianshu.io/upload_images/1801981-56d692d40f4d4edc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二天起床，发现已经安装好了。按照提示，进行初始化操作，比如登录icloud账号，设置用户名等等。然后新的系统就可以用了。

### 日常使用工具安装

这里只按照个人需要，流水账一样的记录下。

- iTerm2 
	
	这个是用来替代Terminal，可以做很多个性化的设置，用起来更顺手。IT必备吧。
	下载地址：http://www.iterm2.com/index.html
	
- oh-my-zsh
	
	默认情况iTerm2和Terminal一样，界面都是比较丑的。这个可以更改iTerm2的主题。同时还集成了各种命令的简写。
	
	下载地址：http://ohmyz.sh
	
- go2shell

	有朋友推荐一些工具来替代Finder，装个这个小工具之后，基本上就懒得再去学一个 类Finder 了。它的功能超级简单，点击一个按钮，在当前位置打开一个Terminal窗口。
	
	下载地址：http://zipzapmac.com/Go2Shell
	（不建议从App Store下载）
	
- homebrew

	这个用来管理软件的，类似wget等。之前安装软件，都是到各家官网去找安装包，自从有了这个，很多软件再也不用到处去找了。
	
	下载地址：http://brew.sh

- alfred

	如果你使用 Spotlight ，alfred可能是一个更好的选择。
	
	下载地址：可以直接在App Store下载。

- Firefox（这里仅仅记录几个常用的插件）
	
	- Adblock Plus 广告已成往事
	- BingDcit 没找到更好的翻译替代工具
	- JSONView 网页查看json数据更友好
	- Octotree 访问Github时，会在左侧列出文件列表，下载浏览更方便
	- User-Agent Switcher 访问一下手机网页会更方便
	- TabTrekker 一个酷炫的桌面插件，每天给你不同的壁纸
	
- Chrome（这里仅仅记录几个常用的插件）
	
	- Adblock Plus
	- JSONView 网页查看json数据更友好
	- Octotree 访问Github时，会在左侧列出文件列表，下载浏览更方便
	- OneTab
	- Pocket
	- Translt 一个翻译插件，双击即可翻译，体验很棒
	- Unsplash Instant 和TabTrekker很像
	
	PS：Chrome插件虽好，但请按需选择，除非你的内存大大的。

- python 和 python3

	mac系统自带了python2.7，这里我还是选择了使用brew安装了python和python3。

- IntelliJ IDEA (Plugins)
	
	- GsonFormat
	- Android ButterKnife Zelezny
	- CodeGlance
	
- 其他开发工具
	
	这个就看各位需要了，我装了一堆。
	
### 收费App

- mweb
- Inboard
- Day One
- 1Password
- Affinity Designer
- macOS Server
- Apple Remote Desktop
- Polarr
- PCalc
- 小历
- Inbox for Gmail
- Paste
- 可意视频
- TODO 还有很多没列出来

别问我是不是装了这里所有软件，告诉你我装了，但不要崇拜哥，万能的淘宝可以实现你的梦想🙈
	
### 环境配置
	
- 环境变量
	
	安装好oh-my-sh之后，就可以添加自己的环境变量配置了，这里简单说些我的配置。首先我选择在一个单独的文件中`~/.bash_profile` 写自己的配置，然后将此文件添加到 `~/.zshrc` 的最后 `source ~/.bash_profile`。`.bash_profile`的内容如下：
		
``` shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home
export GRADLE_HOME=$HOME/android/gradle/gradle-2.14.1
export ANDROID_HOME=$HOME/android/android-sdk-macosx
export NDK_HOME=$ANDROID_HOME/ndk-bundle
export PATH=${PATH}:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/tools:${NDK_HOME}/:${GRADLE_HOME}/bin:${JAVA_HOME}/bin:${JAVA_HOME}/lib:

#
# Aliases
# Create by ttdevs 2016-03-31
#

alias oi='open -a iterm .'
alias of='open -a Finder .'
alias vv='virtualenv'
alias weather='curl http://wttr.in/shanghai'
```


## 0x02 继续装逼

重装系统，我仍是拒绝的。

此文不定期更新。

最后，App Store的软件还是比较贵的，入坑需谨慎 啊 啊 啊

------
[macOS]: http://183.134.9.47/osxapps.itunes.apple.com/apple-assets-us-std-000001/Purple62/v4/af/5f/9d/af5f9d8e-cf9c-8147-c51c-c3c1fececb99/jze1425880974225146329.pkg?wsiphost=local

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


