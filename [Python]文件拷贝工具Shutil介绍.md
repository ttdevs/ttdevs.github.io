
---
title: 「Python」文件拷贝工具Shutil介绍
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Python
categories:
    - 技术
toc: true
cover: cover.jpg 
---



一哥们让帮写个脚本：从一个文件夹中按照指定的规则拷贝部分文件。给半小时时间，虽然水平很菜没有信心，但还是应下了这个需求。


## 0x01 分析

首先想到的就是os，sys这些系统库完成这些操作，由于不熟悉这几个库，还是google一下。不过在搜索的时候发现了这个库： `shutil`。查看了下简直太简单，一行代码完成拷贝。自己之前还想着创建目录，检查文件是否存在，文件读写，各种问题……


## 0x02 shutil

shutil是一个python提供的高级文件操作工具，他可以帮助我们快速的进行常规文件操作。文件拷贝操作如下：

` shutil.copyfile(src, dst, *, follow_symlinks=True) `

第一个参数原文件，第二个参数目标文件位置。简单吧，想想如果java写的话，各种判断，各种iostream，各种蛋疼啊，python就一行啊啊啊啊......

### 方法说明

- copy(src, dst)

    将文件src复制到文件dst，包含权限

- copy2(src, dst)

    同copy，同时复制文件元数据

- copyfile(src, dst)

    将文件src复制到文件dst，但不包含元数据

- copymode(src, dst)

    复制文件权限

- copystat(src, dst)

    将权限位、 最后存取时间、 最后修改时间和标志从src复制到dst

- copytree(src, dst, symlinks=False, ignore=None)

    递归复制目录

- rmtree(path[, ignore_errors[, onerror]])

    删除目录

- move(src, dst)

    递归移动目录

### Demo

``` python
import shutil
    
SOURCE = {
   "1f604",
   ...
   "1f349"}
    
def replace():
   base_path = "drawable/"
   dest_path = "drawable_dest/"
   for name in SOURCE:
       file_name = base_path + "emoji_" + name + ".png"
       dest_name = dest_path + "emoji_" + name + ".png"
       shutil.copy(file_name, dest_name)
    
if __name__ == '__main__':
   replace()
```

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


