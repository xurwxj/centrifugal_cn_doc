# 服务器实现描述

原来的Centrifugal是从[Centrifuge](https://github.com/centrifugal/centrifuge)这个基于Python写的实时消息服务器开始的。然后整个Centrifuge使用Golang进行了重构，重构完成后我们才有了[Centrifugo](https://github.com/centrifugal/centrifugo)。

这个项目的目的是为了接受从应用(web app, mobile app, desktop app)等过来的客户端连接。连接可以是纯的[Websockets](https://developer.mozilla.org/en/docs/WebSockets)协议或者是[SockJS](https://github.com/sockjs/sockjs-client)的方式。Centrifugo保持长连接，发送不同的消息从客户端到服务器 (比如订阅命令，在线状态命令等) 和从服务器发到客户端(比如通道中的新消息，不同的命令反馈等), 在你的应用中通过API请求（比如最常用的发布消息到通道）。
