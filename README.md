## Centrifugo实时消息服务端

![Centrifugal介绍](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/intro.png)

Centrifugal [organization](https://github.com/centrifugal)给你的网站/移动/桌面应用提供一系列工具来增加实时特性。当你需要为你的应用添加实时事件时，它通过整合多个库来带给你开箱即用的体验。

使用指南可以使用你喜爱的语言来创建IM， 实时图表，提醒，各种计数甚至游戏 – [实时消息服务端](server/README.md), [javascript浏览器客户端](client/README.md) 和 [客户端API库](libraries/README.md) 。

Centrifugo服务端与你现有的应用可以轻松整合 - 无须改变你的项目结构和逻辑就可以得到实时事件。 Centrifugo服务端的调用是跨语言的 – 它的API可以使用任何语言来调用，协议是开放并且简单易懂的。我们已经为某些流行语言提供了基础库（看下面），你也可以使用你喜爱的语言自行实现。

你应用中的终端用户可以通过Websocket 或 [SockJS](https://github.com/sockjs/sockjs-client)协议库与Centrifugo服务端通信. SockJS的回溯传输甚至为旧的浏览器或手机浏览器（比如IE 8）提供了实时消息支持。

### 简化的架构

![架构](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/scheme.png)

### 我们的项目

Let's see which projects Centrifugal organization has:

* [centrifugo](https://github.com/centrifugal/centrifugo) - real-time messaging server
    written in Go language
* [centrifuge-js](https://github.com/centrifugal/centrifuge-js) - Javascript client to
    connect to messaging server from web browser.
* [centrifuge-android](https://github.com/centrifugal/centrifuge-android) - Java library to communicate
    with Centrifugo client API over Websockets from Android devices.
* [centrifuge-ios](https://github.com/centrifugal/centrifuge-ios) - Swift library to communicate
    with Centrifugo client API over Websockets from iOS devices.
* [centrifuge-go](https://github.com/centrifugal/centrifuge-go) - Go library to communicate
    with Centrifugo client API over Websockets from Go apps.
* [centrifuge-mobile](https://github.com/centrifugal/centrifuge-go) - Go client and experimental
    bindings generated for iOS and Android using gomobile project.
* [centrifuge-python](https://github.com/centrifugal/centrifuge-go) - Python client for Centrifugo on
    top of asyncio library (work in progress).
* [cent](https://github.com/centrifugal/cent) - Python tools to communicate with Centrifugo API.
* [adjacent](https://github.com/centrifugal/adjacent) - a small wrapper over Cent to
    simplify real-time server integration with Django framework.
* [rubycent](https://github.com/centrifugal/rubycent) - Ruby gem to communicate
    with Centrifugo server API.
* [phpcent](https://github.com/centrifugal/phpcent) - PHP client to communicate
    with Centrifugo server API.
* [gocent](https://github.com/centrifugal/gocent) - Go client to communicate
    with Centrifugo server API.
* [jscent](https://github.com/centrifugal/jscent) - NodeJS client to communicate
    with Centrifugo server API.
* [web](https://github.com/centrifugal/web) - admin web interface for Centrifugo.
    Built on ReactJS.

### Demo

We maintain actual [demo of Centrifugo server instance](https://centrifugo.herokuapp.com) on Heroku (password `demo`).

You can use it to discover Centrifugo without installing it on your computer.

There are 3 endpoints of demo available:

* wss://centrifugo.herokuapp.com/connection/websocket - raw Websocket endpoint
* https://centrifugo.herokuapp.com/connection - SockJS endpoint
* https://centrifugo.herokuapp.com/api/ - HTTP API endpoint

So you can use our client and API libraries to communicate with this demo.

### Examples

Examples can be found [in repo on Github](https://github.com/centrifugal/examples).

At moment we have the following examples:

* django – example shows how to integrate Django application with Centrifugal stack
* Tornado application – shows some general aspects of Centrifugal stack using Tornado server - token generation, private channel signing.
* NodeJS example - shows how to integrate Centrifugo with NodeJS based backend
* WebRTC chat - shows how to use Centrifugo as WebRTC signaling server to create peer-to-peer communication.
* insecure – example shows how to use Centrifugo running in insecure mode without any web application backend.
* jsfiddle – [simplified chat example on jsfiddle](http://jsfiddle.net/FZambia/yG7Uw/) with predefined user ID, timestamp and token which uses Centrifugo [demo](https://centrifugo.herokuapp.com) instance on Heroku

### Community-driven Centrifugo related projects:

* https://github.com/synw/centcli - Terminal client for Centrifugo
* https://github.com/LaraComponents/centrifuge-broadcaster - Broadcast driver for Laravel framework
    to communicate with Centrifugo server API.
* https://github.com/synw/django-instant - Websockets for Django

{% include "SUMMARY.md" %}
