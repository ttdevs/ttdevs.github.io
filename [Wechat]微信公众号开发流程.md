
---
title: 「Wechat」微信公众号开发流程
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Wechat
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x01申请订阅号或者服务号

微信有三种号：订阅号，公众号，服务号，所有人都可以注册订阅号，个人很难注册公众号和服务号；订阅号不能认证，开放的接口也是最少的。既然做开发，这种肯定不好使。因此我们需要选择订阅号或者服务号来做开发。自己没办法注册，可以考虑万能的淘宝。或者我们也可以使用[测试账号](http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)


## 0x02 服务器功能开发

### 配置自己的服务器

在开发之前我们需要进行服务器的配置，这个不涉及具体的语言。登录微信后台，在 开发 > 基本配置 > 服务器配置 中进行配置。这里需要配置四个参数：

- URL

	这个是我们服务器的地址，必须是80端口。微信会把所有的事件和消息等都推送到这个地址上。
	
- Token

	这个Token是我们分配给微信的

- EncodingAESKey
	
	微信消息的加密秘钥
	
- 消息加解密方式

	没有特殊要求我们可以选择明文模式
	
更多信息可以参考[这里](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319&token=&lang=zh_CN)。

### 微信服务器验证

如果我们直接配置了上面的参数，是无法保存的，只有在我们配置的地址正确处理了微信的验证请求之后才可以保存成功。过程是这样的，微信会以get方式向我们配置的地址发一个请求，这个请求带了验证信息，和返回的字符串，他们分别是 `signature`、 `timestamp`、 `nonce`、 `echostr`，前三个参数用于验证，后一个用于返回微信。如果我们按照规则验证参数是正确的（这里会用到我们配置的Token），将echostr字段的信息返回，这样就可以保存了。当然，你也可以不做验证直接返回echostr参数内容也是可以，但是不推荐这样做。更多信息可以参考[这里](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319&token=&lang=zh_CN)。

### 微信事件处理

配置了上面的参数我们就可以处理微信发给我们的各种信息了。微信所有信息都是以POST的方式发送到我们之前配置的地址上。在URL参数中会携带四个参数`signature`、 `timestamp`、 `nonce`和`openid`，数据部分在body中。按照微信的接口协议，我们就可以开始微信功能开发了。

最后，无论是实用的是什么语言，都建议你去找一份封装的SDK来使用。这样可以大大的增加你的开发效率，当然，如果你是像我一样为了学习，可以自己去解析这些xml格式的消息～～

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

