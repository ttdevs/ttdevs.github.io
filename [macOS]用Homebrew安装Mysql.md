
---
title: 「macOS」用Homebrew安装Mysql
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


## 0x00 需求

简单记录mac下mysql安装。这里需要一个前提——先安装 [`homebrew`](http://brew.sh/index_zh-cn.html)，具体可以参考[这里](http://brew.sh/index_zh-cn.html)。


##  0x01 安装

``` shell
➜  ~ brew install mysql
＃ 以下为删除上个版本的遗留
➜  ~ cd /usr/local/var/mysql
➜  mysql git:(master) ls
auto.cnf           ib_buffer_pool     mysql              server-cert.pem
ca-key.pem         ib_logfile0        mysqld_safe.pid    server-key.pem
ca.pem             ib_logfile1        performance_schema sys
client-cert.pem    ibdata1            private_key.pem    ttdevs.local.err
client-key.pem     ibtmp1             public_key.pem     ttdevs.local.pid
➜  mysql git:(master) rm -rf ttdevs.local.err
➜  mysql git:(master) cd ~
＃ 以上为删除上个版本的遗留
```

> 卸载： `brew uninstall mysql`


## 0x02 配置

``` shell
➜  ~ /usr/local/opt/mysql/bin/mysql_secure_installation

Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No:
Please set the password for root here.

New password:

Re-enter new password:
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done!
➜  ~ mysql.server start
Starting MySQL
 SUCCESS!
```


## 0x03 启动停止

- `brew services start/stop mysql ` 
- `mysql.server start/stop`

两种方式都可以，但是不可以交叉使用，比如用 `brew services start mysql` 启动 却不能用 `mysql.server stop` 停止。


## 0x04 GUI客户端

- [Sequel Pro](http://www.sequelpro.com)

	免费

- [Navicat For Mysql](https://www.navicat.com.cn/)

	收费


## 0x05 简单命令纪录

- 登录：`mysql -u username -p`
- 推出：`exit;`
- 默认 `data` 目录：`/usr/local/var/mysql`
- Emoji表情的支持请使用编码：`utf8mb4`
- TODO 2016-09-07


## 0xFF 参考

1. https://segmentfault.com/q/1010000000475470
2. http://pein0119.github.io/2015/03/25/MySQL%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%AF%E5%8A%A8%E9%94%99%E8%AF%AF-The-server-quit-without-updating-PID-file/

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

