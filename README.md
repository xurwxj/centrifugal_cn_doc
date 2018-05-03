## Centrifugo实时消息服务端

![Centrifugal介绍](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/intro.png)

Centrifugal [organization](https://github.com/centrifugal)给你的网站/移动/桌面应用提供一系列工具来增加实时特性。当你需要为你的应用添加实时事件时，它通过整合多个库来带给你开箱即用的体验。

使用指南可以使用你喜爱的语言来创建IM， 实时图表，提醒，各种计数甚至游戏 – [实时消息服务端](server/README.md), [javascript浏览器客户端](client/README.md) 和 [客户端API库](libraries/README.md) 。

Centrifugo服务端与你现有的应用可以轻松整合 - 无须改变你的项目结构和逻辑就可以得到实时事件。 Centrifugo服务端的调用是跨语言的 – 它的API可以使用任何语言来调用，协议是开放并且简单易懂的。我们已经为某些流行语言提供了基础库（看下面），你也可以使用你喜爱的语言自行实现。

你应用中的终端用户可以通过Websocket 或 [SockJS](https://github.com/sockjs/sockjs-client)协议库与Centrifugo服务端通信. SockJS的回溯传输甚至为旧的浏览器或手机浏览器（比如IE 8）提供了实时消息支持。

### 简化的架构

![架构](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/scheme.png)

### 我们的项目

Centrifugal开发的项目有以下:

* [centrifugo](https://github.com/centrifugal/centrifugo) - Go语言实现的实时消息服务端
* [centrifuge-js](https://github.com/centrifugal/centrifuge-js) - 浏览器端用Javascript实现的连接centrifugo库
* [centrifuge-android](https://github.com/centrifugal/centrifuge-android) - Android设备基于Java实现Websocket的连接centrifugo库
* [centrifuge-ios](https://github.com/centrifugal/centrifuge-ios) - IOS设备基于Swift 实现Websocket的连接centrifugo库
* [centrifuge-go](https://github.com/centrifugal/centrifuge-go) - 基于Go语言实现Websocket的连接centrifugo库
* [centrifuge-mobile](https://github.com/centrifugal/centrifuge-go) - 基于Go语言gomobile项目实现Websocket的连接centrifugo库，可用于IOS和Android
* [centrifuge-python](https://github.com/centrifugal/centrifuge-go) - 基于Python异步库实现Websocket的连接centrifugo库（正在开发中）
* [cent](https://github.com/centrifugal/cent) - Python工具，使用Centrifugo API
* [adjacent](https://github.com/centrifugal/adjacent) - 基于Cent的小工具用于Django中简单整合实时消息服务端
* [rubycent](https://github.com/centrifugal/rubycent) - 基于Ruby和Centrifugo server API实现的库
* [phpcent](https://github.com/centrifugal/phpcent) - 基于PHP和Centrifugo server API实现的库
* [gocent](https://github.com/centrifugal/gocent) - 基于Go语言和Centrifugo server API实现的库
* [jscent](https://github.com/centrifugal/jscent) - 基于NodeJS和Centrifugo server API实现的库
* [web](https://github.com/centrifugal/web) - Centrifugo的管理网站，基于ReactJS开发

### 演示站点

我们在Heroku上建立了[Centrifugo服务演示实例和管理站点](https://centrifugo.herokuapp.com)(密码是 `demo`).

你可以在安装到你的环境之前先体验Centrifugo。

演示站点提供3种客户端连接方式，不管是用客户端还是API库都可以连接:

* wss://centrifugo.herokuapp.com/connection/websocket - Websocket连接地址
* https://centrifugo.herokuapp.com/connection - SockJS连接地址
* https://centrifugo.herokuapp.com/api/ - HTTP API连接地址

### 例子

可以在[Github](https://github.com/centrifugal/examples)上找到代码样例.

目前我们提供了以下代码样例:

* django – 展示如何整合Django和Centrifugal
* Tornado应用 – 展示一些常用方法来整合Centrifugal和Tornado - token生成, 私有channel登录
* NodeJS例子 - 展示如何整合Centrifugo和NodeJS后端
* WebRTC聊天 - 展示如何把Centrifugo作为WebRTC signaling server来创建点对点通信
* insecure – 展示如何在不安全的环境下独立运行Centrifugo
* jsfiddle – [jsfiddle的聊天样例](http://jsfiddle.net/FZambia/yG7Uw/)，使用部署在Heroku[demo](https://centrifugo.herokuapp.com) 上的Centrifugo，预定义的用户ID， 时间戳和token 

### 社区开发的与Centrifugo相关的项目:

* https://github.com/synw/centcli - Centrifugo终端客户端
* https://github.com/LaraComponents/centrifuge-broadcaster - Laravel框架整合Centrifugo server API的广播驱动
* https://github.com/synw/django-instant - Django的Websockets实现 

{% include "SUMMARY.md" %}
