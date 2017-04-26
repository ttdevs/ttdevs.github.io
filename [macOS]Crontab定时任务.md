## 0x00 crontab

我们可以使用crontab执行一些周期任务

## 0x01 crontab命令

crontab [-u user] file

crontab [-u user] { -e | -l | -r }


- `crontab file_name`

将file做为crontab的任务列表文件并载入crontab

- `crontab -e`

编辑crontab文件内容

- `crontab -l`

显示crontab文件内容

- `crontab -r`

删除载入后的crontab文件内容

## 0x02 crontab文件

- `分` `时` `日` `月` `星期` `要运行的命令`

    - 第1列 分钟1～59
    - 第2列 小时1～23（0表示子夜）
    - 第3列 日1～31
    - 第4列 月1～12
    - 第5列 星期0～7（0,7表示星期天）
    - 第6列 要运行的命令

- 几个特殊符号：`/` `-` `,`
    
    - `/` 每(per): `*/2` 每两个单位
    - `-` 连续: `1-5` 1,2,3,4,5
    - `,` 非连续: `1,2,5` 1和2和5

## 0x03 Demo


``` shell
# 每分钟向用户目录下的log.txt写入当时日期
* * * * * /bin/date >> ~/log.txt 

# 每分钟
* * * * * /bin/date >> ~/log.txt

# 每5分钟
*/5 * * * * /bin/date >> ~/log.txt

# 每小时的第五分钟
5 * * * * /bin/date >> ~/log.txt 

# 4月1日早上8点
0 8 1 4 * /bin/date >> ~/log.txt 

# 4和5月 1日早上8点
0 8 1 4,5 * /bin/date >> ~/log.txt 

# 12306
23-7 * * * * /bin/date >> ~/log.txt 
```

# 更新(2015-03-26)

## 0x01 crontab辅助工具

这个工具可以生成crontab文件。比如我们按照自己的想法选择了任务的时间，可以使用它帮助我们生成对应的crontab文件。

地址：http://www.crontab-generator.org/

## 0x02 crontab的调试

这个建议在命令的后面加上 `... >> ~/log.txt 2>&1`，这样我们就可以在用户目录下看到crontab执行的日志了。如：

``` shell
* * * * * /bin/date >> ~/log.txt >> ~/log.txt 2>&1
```

## 0x03 环境变量

虽然我们知道crontab执行时是没有环境变量的，但是这个确是个讨厌的东西。我们在本地调试好了相关的代码，用crontab却怎么执行也不正确，比如执行下面的shell：

``` shell
#!/bin/bash
py3=$(which python3)
echo py3
...
```
默认情况什么也不会输出。下面的逻辑自然就是错的了。为了解决这个问题，我们可以在最开始加上这么一行 `source /etc/profile` ，然后在执行就可以顺利拿到我们的值了：

``` shell
#!/bin/bash
source /etc/profile
py3=$(which python3)
echo py3
...
```

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

