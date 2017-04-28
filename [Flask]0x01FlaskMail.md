
---
title: 「Flask」0x01FlaskMail
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Flask
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x00  简介

> Welcome to Flask-Mail, you can use to send mail in your web site.

Flask-Mail 提供了一个简单的接口，让我们可以方便的在 Flask 应用中使用 SMTP协议 发送邮件。


## 0x01 安装

`pip install Flask-Mail`

目前最新版本是 0.9.1 ，从 [Flask-Mail的github地址](https://github.com/mattupstate/flask-mail)可以发现，两年没有更新了。不过这个并不影响我们使用，毕竟发送邮件是个比较成熟的东西，只祈求和最新的Flask不要出现兼容问题即可。

如果你不想使用这个工具，可以找找其他的，不过你会发现，其他的可能更老，也是醉了=_=

如果时间充足，我们可以去读读 Flask-Mail 的源码，你会发现，最终是用系统自带的 smtplib 实现的。


## 0x02 使用

一句话总结：Flask-Mail 的使用还是相当easy的。

### 配置

Flask-Mail 使用 Flask 标准的配置 API 进行配置。下面是所有的配置选项：

- MAIL_SERVER : 默认为 '127.0.0.1'
- MAIL_PORT : 默认为 25
- MAIL_USE_TLS : 默认为 False
- MAIL_USE_SSL : 默认为 False
- MAIL_DEBUG : 默认为 app.debug
- MAIL_USERNAME : 默认为 None
- MAIL_PASSWORD : 默认为 None
- MAIL_DEFAULT_SENDER : 默认为 None
- MAIL_MAX_EMAILS : 默认为 None
- MAIL_SUPPRESS_SEND : 默认为 app.testing
- MAIL_ASCII_ATTACHMENTS : 默认为 False

这个配置的参数我们可以在[这里](https://github.com/mattupstate/flask-mail/blob/master/flask_mail.py)看到。

### 初始化

可以使用下面的两种方式进行初始化：

- 第一种方式：

    使用传入到 Mail 实例中的应用程序的配置项进行邮件发送

    ``` python
    from flask import Flask
    from flask_mail import Mail
    
    app = Flask(__name__)
    mail = Mail(app)
    ```

- 第二种方式：

    使用 Flask 的 current_app 中的配置项进行邮件发送，如果我们有多个 不同配置的应用程序 则使用此种方式比较方便

    ``` python
    mail = Mail()
    
    app = Flask(__name__)
    mail.init_app(app)
    ```

### 发送

发送之前我们需要先构建一个 Message 对象，如下：

``` python
from flask_mail import Message

msg = Message("Hello Flask", sender="ttdevs@gmail.com",  recipients=["ttdevs@live.com"])
```

我们也可以同时指定多个收件人

``` python
msg.recipients = ["ttdevs@gmail.com", "ttdevs@foxmail.com"]
msg.add_recipient("ttdevs@live.com")
```

如果我们配置了 `MAIL_DEFAULT_SENDER` 字段，就可以不再设置 sender ，这个时候会使用  `MAIL_DEFAULT_SENDER` 中指定的发件人，像这样：

``` python
msg = Message("Hello Flask", recipients=["ttdevs@live.com"])
```

如果我们希望在收件列表中显示一个名字，可以通过一个二元祖来指定：

``` python
msg = Message("Hello", sender=("ttdevs", "ttdevs@live.com"))
```

同时， 我们还可以指定下面两个字段：

``` python
msg.body = "this is body string"
msg.html = "<h2>this is html message</b>"
```

最后就是发送：

``` python
mail.send(msg)
```

发送完毕后，与邮件服务器的链接就会关闭。

### 发送大量邮件

如果我们一次发送大量的邮件，可以通过下面的方式发送：

``` python
with mail.connect() as conn:
    for user in users:
        message = '...'
        subject = "hello, %s" % user.name
        msg = Message(recipients=[user.email],
                      body=message,
                      subject=subject)
        conn.send(msg)
```

与电子邮件服务器的连接会一直保持直到所有的邮件都已经发送完毕才会断开。

>Some mail servers set a limit on the number of emails sent in a single connection. You can set the max amount of emails to send before reconnecting by specifying theMAIL_MAX_EMAILS setting.

### 添加附件

在邮件中添加附件同样非常简单：

``` python
image = 'umbrella.jpg'
with app.open_resource(image) as fp:
    msg.attach(image, 'image/jpg', fp.read())
```


## 0x03 Demo

下面是一个简单的Demo，装好相关的类库， 直接可以跑，大家可以参考：

``` python
#!/usr/bin/env python3
# coding:utf-8

from flask import Flask
from flask_mail import Mail
from flask_mail import Message

app = Flask(__name__)
app.config['MAIL_SERVER'] = 'smtp.qq.com'
app.config['MAIL_PORT'] = 465
app.config['MAIL_USE_SSL'] = True
app.config['MAIL_USERNAME'] = 'iot.raspi@qq.com'
app.config['MAIL_PASSWORD'] = '********'
app.config['MAIL_DEFAULT_SENDER'] = 'iot.raspi@qq.com'

mail = Mail(app)


@app.route('/')
def welcome():
    msg = Message('这是一封测试邮件Header', recipients=['56532799@qq.com'])
    msg.body = '这是一封测试邮件 bodyer'
    msg.html = '这是一封测试邮件 htmler'
    image = 'umbrella_伞.jpg'
    with app.open_resource(image) as fp:
        msg.attach(image, 'image/jpg', fp.read())
    mail.send(msg)
    return 'Hello world!'


if __name__ == '__main__':
    app.run(debug=tuple)
```

### 邮件参数说明

上面的Demo中用到QQ邮箱，这里简单介绍下。由于QQ邮箱需要安全验证，所以我们配置下面几个参数：
    
- MAIL_SERVER

    SMTP地址：smtp.qq.com

- MAIL_PORT

    由于需要安全验证，所以此处用465
    
- MAIL_USE_SSL：True

    SSL，走加密方式

- MAIL_USERNAME
    
    iot.raspi@qq.com
    
- MAIL_PASSWORD
    
    这个地方我们需要使用授权码：
    打开 `QQ邮箱` > `设置` > `POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务` > `生成授权码`
    
最后别忘了打开SMTP服务～～


## 0x04 其他 

（TODO 留坑，其实暂时还没学到～～）

### 单元测试

### 禁止发送邮件

### 头注入

### 信号量

### API


## 0xFF 参考

1. http://pythonhosted.org/Flask-Mail/
2. http://www.pythondoc.com/flask-mail/
3. https://pypi.python.org/pypi/Flask-Mail/

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)



