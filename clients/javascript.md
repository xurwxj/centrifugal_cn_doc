Javascript浏览器客户端
=========================

到目前为止，您已经了解Centrifugo服务器是如何实现及工作的。现在可以通过web浏览器连接你的web应用用户到Centrifugo服务器了。

Js客户端就是为了这一目的而存在的。

* [安装与快速开始](#install-and-quick-start)
* [连接参数](#connection-parameters)
* [配置参数](#configuration-parameters)
* [客户端API](#client-api)
* [私有通道](#private-channels)
* [连接检测](#connection-check)

Js客户端库源代码位于[repo on Github](https://github.com/centrifugal/centrifuge-js).

Js客户端库可以有2种方式连接服务器：使用纯粹的Websocket或是[SockJS](https://github.com/sockjs/sockjs-client)库（当浏览器不支持Websocket的时候可以通过它来实现各种回调）。

使用Js客户端你可以：

* 把你的用户连接到实时通信服务器
* 订阅通道并且监听所有该通道的新消息
* 获取通道的在线信息（所有当前订阅该通道的客户端）
* 获取通道的历史消息
* 接收通道中加入/离开的事件（当有人订阅或取消订阅的时候）
* 推送新的消息到通道

*注意：为了获取通道的在线信息、历史消息、事件、推送，你必须要对通道进行相应的设置

如果你在寻找旧的API文档 (`centrifuge-js` < 1.3.0) - [你可以在这里查看](https://github.com/centrifugal/documentation/tree/c69ca51f21c028a6b9bd582afdbf0a5c13331957/client)


## 安装与快速开始

使用Js客户端库的最简单方式是在网页中使用`script` 标签:

```html
<script src="centrifuge.js"></script>
```

从[库](https://github.com/centrifugal/centrifuge-js)中下载.

浏览器客户端也可以通过`npm` 和 `bower`安装:

```bash
npm install centrifuge
```

或:

```bash
bower install centrifuge
```

如果你想使用SockJS，你必须要在centrifuge.js引入SockJS

```html
<script src="//cdn.jsdelivr.net/sockjs/1.1/sockjs.min.js" type="text/javascript"></script>
<script src="centrifuge.js" type="text/javascript"></script>
```

**如果你想支持ie8以下版本** 你必须要引入JSON polyfill库:

```html
<script src="//cdnjs.cloudflare.com/ajax/libs/json3/3.3.2/json3.min.js" type="text/javascript"></script>
<script src="//cdn.jsdelivr.net/sockjs/1.1/sockjs.min.js" type="text/javascript"></script>
<script src="centrifuge.js" type="text/javascript"></script>
```

当你引入所有需要的库后，你可以创建新的`Centrifuge`对象实例、订阅通道并且调用`.connect()`方式开始实际的连接:

```javascript
<script type="text/javascript">

    var centrifuge = new Centrifuge({
        url: 'http://centrifuge.example.com/connection',
        user: "USER ID",
        timestamp: "UNIX TIMESTAMP SECONDS",
        token: "SHA-256 HMAC TOKEN"
    });

    centrifuge.subscribe("news", function(message) {
        console.log(message);
    });

    centrifuge.connect();

</script>
```

在上面的例子中初始化`Centrifuge`对象实例，订阅了通道`news`, 打印所有收到的新消息并且连接到了Centrifugo服务器。这就是所有从客户端连接到实时服务器需要的代码!

***`Centrifuge`对象是一个[EventEmitter](https://github.com/Olical/EventEmitter/blob/master/docs/api.md)对象.***

参数 `url`, `user`, `timestamp` 和 `token`是必须的，我们来看看这些参数及其它参数介绍。

## 连接参数

在上面我们已经介绍初始化`Centrifuge`对象你必须要提供参数: `url`, `user`, `timestamp`, `token`, 以及可选的 `info`.

**注意所有连接参数除url外必须从后端输出**. 你可以通过服务器端渲染、Cookie或AJAX请求来提供`user`,`timestamp`, `info` 和 `token`.

下面来看每个参数的含义：

#### url (必须)

`url` – 是Centrifugo的连接地址.

如果你的Centrifugo使用了域名 `centrifugo.example.com`，那么连接地址为:

* SockJS连接地址是 `http://centrifugo.example.com/connection`
* 纯粹的Websocket连接地址是 `ws://centrifugo.example.com/connection/websocket`

如果你的Centrifugo启用了SSL(**建议启用**)，那边连接地址就要必须以`https` (SockJS)或 `wss` (Websocket)开始。

你也可以设置 `url`是`http://centrifugo.example.com`， Js库会根据你的引入自动检测连接地址并使用SockJS或 Websocket来连接。

#### user (必须)

`user` **字符串**是你web应用的当前用户ID。**它可以为空，如果用户未登录，但这个时候你必须要为通道启用** `anonymous`接入许可（在Centrifugo中设置`anonymous: true`）。

注意, **它必须是字符串类型**，哪怕你的应用使用数字作为用户ID，也要转换成字符串。

#### timestamp (必须)

`timestamp`字符串是连接生成的以秒为单位的UNIX服务器时间。

注意，大部分语言默认把UNIX时间戳返回为Float类型值或是包含了毫秒。Centrifugo服务器**仅需要以秒为单位的UNIX服务器时间字符串类型**。比如Python就可以通过使用`"%.0f" % time.time()`或`str(int(time.time()))`来得到类似`"1451991486"`的正确值。

#### token (必须)

`token`是你的web应用基于Centrifugo`secret` key, `user` ID, `timestamp` (和可选的`info` - 看下面)生成的加密串。具体可以查看 [Tokens和签名](../server/tokens_and_signatures.md).

**对于Python, Ruby, NodeJS, Go 和 PHP，我们已经在库中提供现成的方法来生成**.

正确的token保证了有效的用户ID和时间戳来请求Centrifugo服务器。Token类似于HTTP cookie，客户端必须不要暴露出来，特别是*使用私有通道*的时候。

#### info (可选)

当连接Centrifugo的时候，你可以额外可选的提供参数`info`:

```javascript
var centrifuge = new Centrifuge({
    url: 'http://centrifuge.example.com/connection',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    info: '{"first_name": "Alexandr", "last_name": "Emelin"}',
    token: 'TOKEN'
});
```

`info` 是连接的额外信息，必须是 **有效编码的JSON字符串**. 但如果要防止客户端发送错误的`info`， **这个JSON字符串是在生成token过程中必须的**.

如果你不想使用`info` - 在连接Centrifugo的时候忽略这个参数即可，但要注意如果不使用的话，在生成token过程中不要同样不要使用(`info`必须是空字符串) .

## 配置参数

请参照可选的初始化`Centrifuge`对象实例参数.

#### transports

使用SockJS时的额外配置参数：`transports`. 它定义了允许的SockJS协议及默认匹配

```javascript
var centrifuge = new Centrifuge({
    ...
    transports: [
        'websocket', 'xdr-streaming', 'xhr-streaming',
        'eventsource', 'iframe-eventsource', 'iframe-htmlfile',
        'xdr-polling', 'xhr-polling', 'iframe-xhr-polling', 'jsonp-polling'
    ]
});
```

上述代码列出了SockJS 所有可用的协议.

如果`centrifuge-js`仅使用`websocket` 和 `xhr-streaming`协议:

```javascript
var centrifuge = new Centrifuge({
    url: 'http://centrifuge.example.com/connection',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    info: '{"first_name": "Alexandr", "last_name": "Emelin"}',
    token: 'TOKEN',
    transports: ["websocket", "xhr-streaming"]
});
```

#### sockJS

**1.3.7版本中新出**. `sockJS`选项允许提供SockJS客户端对象给Centrifuge客户端.

比如ES6下.

```javascript
import Centrifuge from 'centrifuge'
import SockJS from 'sockjs-client'

var centrifuge = new Centrifuge({
    url: 'http://centrifuge.example.com/connection',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    info: '{"first_name": "Alexandr", "last_name": "Emelin"}',
    token: 'TOKEN',
    sockJS: SockJS
});
```

#### debug

`debug`是一个布尔值选项，默认是`false`. 启用后，一系列调试信息就会输出到控制台，对于问题定位和解决有比较大的帮助.

#### insecure

`insecure` 是一个布尔值选项，默认是`false`. 启用后，客户端将可以在非安全模式中连接服务器，可以阅读[特定章节](../mixed/insecure_modes.md).

这个选项对于需要快速连接Centrifugo建立演示时非常有用，它允许无须`token`, `timestamp` 和 `user`连接Centrifugo. 更多信息请阅读[特定章节](../mixed/insecure_modes.md).

#### retry

当客户端断开连接的时候，它可以自动重连，`retry`这个参数可以设置最小的重连尝试时间，以毫秒为单位，默认是`1000`毫秒.

#### maxRetry

`maxRetry` 设置最大的重连尝试时间，以毫秒为单位，当超过这个时间后，客户端将不再尝试重连，默认是`20000`毫秒.

#### resubscribe

`resubscribe` 是一个布尔值选项，可以用来禁用自动重新订阅，默认是`true`。比如你不需要的自动控制重新订阅并且不等`connect`事件触发（首次或重连的时候）就开始订阅. `centrifuge-js`默认在连接建立时会自动重新订阅.

#### server

`server` 是SockJS特定的选项，允许在连接urls中设置服务器名字来替代默认的随机字符，详情请查看SockJS文档.

#### authEndpoint

`authEndpoint`授权验证url，用于订阅私有通道时发送订阅验证请求，默认是`/centrifuge/auth/`. 同时请注意以下相关的有用选项:

* `authHeaders` - 验证请求时的头信息map (默认值是 `{}`)
* `authParams` - 验证请求url中的参数map (默认值是 `{}`)
* `authTransport` - 验证请求的协议 (默认值是 `ajax`, 另外一个可选值是 `jsonp`)

#### refreshEndpoint

`refreshEndpoint` Centrifugo连接检测机制启用时的刷新客户端连接参数url. 与其相关的选项有:

* `refreshHeaders` - 刷新请求的头信息map (默认值是 `{}``)
* `refreshParams` - 刷新请求url中的参数map (默认值是 `{}`)
* `refreshTransport` - transport to use for refresh request (默认值是 `ajax`, 另外一个可选值是 `jsonp`)
* `refreshData` - AJAX POST请求时额外发送的信息(作为 JSON 格式).
* `refreshAttempts` - 刷新请求的重试次数 (默认值是 `null` - 无限)
* `refreshFailed` - 当请求达到 `refreshAttempts`设置的限制时的处理回调. 默认值是 `null` - 不进行处理.

## 客户端API

当 `Centrifuge`对象初始化后就可以开始与服务器通信了.

#### 连接方法

如之前所讲，调用`connect()`方法来连接Centrifugo:

```javascript
var centrifuge = new Centrifuge({
    // ...
});

centrifuge.connect();
```

`connect()` 通过初始化过程中指定的连接参数和配置选项进行调用.

#### 连接事件

当连接建立后，你可以监听`connect`事件:

```javascript
centrifuge.on('connect', function(context) {
    // now client connected to Centrifugo and authorized
});
```

`context`的内容如下面例子所示类似:

```javascript
{
    client: "79ec54fa-8348-4671-650b-d299c193a8a3",
    transport: "raw-websocket",
    latency: 21
}
```

* `client` – Centrifugo给予的本连接客户端ID (字符串格式)
* `transport` – 连接建立的协议名称 (字符串格式)
* `latency` – 建立的持续时长 (整数格式，毫秒). 从发送连接请求到得到响应的时长，1.3.1新出的值.

#### 断开事件

`disconnect`是每次客户端断开连接时触发，有可能是网络断开或是被Centrifugo服务器断开.

```javascript
centrifuge.on('disconnect', function(context) {
    // do whatever you need in case of disconnect from server
});
```

`context`的内容如下面例子所示类似:

```javascript
{
    reason: "connection closed",
    reconnect: true
}
```

* `reason` – 断开的原因 (字符串格式)
* `reconnect` – 客户端将是否重连 (布尔值格式)


#### 错误事件

`error`在得到错误响应的时候触发，正常是不会出现的，但对于记录异常的问题非常有用，同时对于订阅、`publish`, `presence`, `history`调用错误时可以有助于重试或重新订阅.

```javascript
centrifuge.on('error', function(error) {
    // handle error in a way you want, here we just log it into browser console.
    console.log(error)
});
```

`error`的内容如下面例子所示类似:

```javascript
{
    "message": {
        "method": "METHOD",
        "error": "ERROR DESCRIPTION",
        "advice": "OPTIONAL ERROR ADVICE"
    }
}
```

`message` – 服务器返回的消息，包含了`error`字段. 


#### 断开连接的方法

当你需要断开与服务器的连接时，调用`disconnect`:

```javascript
centrifuge.disconnect();
```

注意，调用这个方法后将不会再自动重连，需要你手工调用`connect`方法去重新建立连接.


## 订阅

下面讲如何接收实时消息


### 订阅方法

要订阅一个通道，我们必须使用`subscribe`方法来获取新的消息:

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle new message coming from channel "news"
    console.log(message);
});
```

就这么简单！可同时处理在订阅时可能的事件:

* `message` – 新消息接收到时的事件调用
* `join` – 有人加入通道时调用
* `leave` – 有人离开通道时调用
* `subscribe` – 订阅通道成功时，这个可以被多次调用.
* `error` – 订阅失败时的错误处理调用.
* `unsubscribe` – 取消订阅时的调用

大部分情况下，你只需要处理部分事件即可，下面看一下各种事件的处理格式. 有2种设置回调的方式来处理事件，一是提供回调对象作为`subscribe` 方法的第2个参数.

```javascript
var callbacks = {
    "message": function(message) {
        // See below description of message format
        console.log(message);
    },
    "join": function(message) {
        // See below description of join message format
        console.log(message);
    },
    "leave": function(message) {
        // See below description of leave message format
        console.log(message);
    },
    "subscribe": function(context) {
        // See below description of subscribe callback context format
        console.log(context);
    },
    "error": function(errContext) {
        // See below description of subscribe error callback context format
        console.log(err);
    },
    "unsubscribe": function(context) {
        // See below description of unsubscribe event callback context format
        console.log(context);
    }
}

var subscription = centrifuge.subscribe("news", callbacks);
```

另外一个是使用 `on`方法来处理:

```javascript
var subscription = centrifuge.subscribe("news");

subscription.on("message", messageHandlerFunction);
subscription.on("subscribe", subscribeHandlerFunction);
subscription.on("error", subscribeErrorHandlerFunction);
```

***订阅对象是[EventEmitter]的实例(https://github.com/Olical/EventEmitter/blob/master/docs/api.md).***


### 订阅的加入和离开事件

可以掌控通道订阅的`join` 和 `leave` 事件：

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
}).on("join", function(message) {
    console.log("Client joined channel");
}).on("leave", function(message) {
    console.log("Client left channel");
});
```


### 订阅事件上下文内容格式

下面来看一下订阅事件各种回调的上下文内容格式.

#### format of message event context

新消息的事件上下文内容格式:

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "channel":"$public:chat",
    "data":{"input":"hello"},
}
```

I.e. `data`包含了实际发送的内容. 消息有可能多包含`client` 字段 (客户端ID):

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "channel":"$public:chat",
    "data":{"input":"hello"},
    "client":"7080fd2a-bd69-4f1f-6648-5f3ceba4b643"
}
```

也有可能多包含了`info`字段，特别是这个消息是由js客户端直接使用`publish` 方法发送时:

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "info":{
        "user":"2694",
        "client":"7080fd2a-bd69-4f1f-6648-5f3ceba4b643",
        "default_info":{"name":"Alexandr"},
        "channel_info":{"extra":"extra JSON data when authorizing private channel"}
    },
    "channel":"$public:chat",
    "data":{"input":"hello"},
    "client":"7080fd2a-bd69-4f1f-6648-5f3ceba4b643"
}
```


#### 加入/离开事件上下文内容格式

I.e. `on("join", function(message) {...})` or `on("leave", function(message) {...})`

```javascript
{
    "channel":"$public:chat",
    "data":{
        "user":"2694",
        "client":"2724adea-6e9b-460b-4430-a9f999e94c36",
        "default_info":{"first_name":"Alexandr"},
        "channel_info":{"extra":"extra JSON data when authorizing"}
    }
}
```

`default_info` 和 `channel_info` 只有在不为空的时候才有.


#### 订阅事件上下文内容格式

I.e. `on("subscribe", function(context) {...})`

```javascript
{
    "channel": "$public:chat",
    "isResubscribe": true
}
```

`isResubscribe` – 表示是首次订阅 (`false`) 还是重新订阅 (`true`)


#### 订阅错误事件上下文内容格式

I.e. `on("error", function(err) {...})`

```javascript
{
    "error": "permission denied",
    "advice": "fix",
    "channel": "$public:chat",
    "isResubscribe": true
}
```

`error` - 错误描述
`advice` - 可选的建议 (`retry` 或 `fix` )
`isResubscribe` – 表示是首次订阅 (`false`) 还是重新订阅 (`true`)


#### 取消订阅事件上下文内容格式

I.e `on("unsubscribe", function(context) {...})`

```
{
    "channel": "$public:chat"
}
```

I.e. 只有 `channel` 这一项内容.


### 订阅的在线状态方法

`presence`允许获取当前订阅该通道的客户端信息，注意只有相应设置启用时才能获取.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.presence().then(function(message) {
    // presence data received
}, function(err) {
    // presence call failed with error
});
```

`presence`以promise方式，如上述代码一样使用，成功时的`message`格式内容为:

```javascript
{
    "channel":"$public:chat",
    "data":{
        "2724adea-6e9b-460b-4430-a9f999e94c36": {
            "user":"2694",
            "client":"2724adea-6e9b-460b-4430-a9f999e94c36",
            "default_info":{"first_name":"Alexandr"}
        },
        "d274505c-ce63-4e24-77cf-971fd8a59f00":{
            "user":"2694",
            "client":"d274505c-ce63-4e24-77cf-971fd8a59f00",
            "default_info":{"first_name":"Alexandr"}
        }
    }
}
```

错误回调中的`err`的内容格式:

```javascript
{
    "error": "timeout",
    "advice": "retry"
}
```

* `error` – 错误描述 (字符串格式))
* `advice` – 处理建议 (字符串格式, 当前只有"fix"或 "retry")


### 订阅历史方法

`history`方法允许获取通道中的最新消息，但注意一定要在Centrifugo配置进行设置.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.history().then(function(message) {
        // history messages received
    }, function(err) {
        // history call failed with error
    });
});
```

成功后的回调`message`格式为:

```javascript
{
    "channel": "$public:chat",
    "data": [
        {
            "uid": "87219102-a31d-44ed-489d-52b1a7fa520c",
            "channel": "$public:chat",
            "data": {"input": "hello2"}
        },
        {
            "uid": "71617557-7466-4cbb-760e-639042a5cade",
            "channel": "$public:chat",
            "data": {"input": "hello1"}
        }
    ]
}
```

`data`是通道中的消息数组.其它额外的字段，比如 `client`, `info`这些如果在原始消息中的字段也可以包括在其中.

`err`格式是跟`presence` 方法一样的.


### 订阅后的发布方法

`publish`允许客户端在订阅通道后直接发布消息到通道中。Centrifugo本身的主旨是服务器端的push。允许客户端发布可以在某些场景中贴合实际，并有一定的实时效果.

**要特别强调的：使用客户端发布并非推荐的Centrifugo使用方式，并不是生产推荐的方式，只是为了某些场景提供便利而已，在众多实时应用中，新消息应该先发到后端再通过Centrifugo发布.**

当在客户端使用`publish`方法时，你的应用不会收到这些消息，除非你制定特别的逻辑.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.publish({"input": "hello world"}).then(function() {
        // success ack from Centrifugo received
    }, function(err) {
        // publish call failed with error
    });
});
```

`err`与`presence` 方法一样的格式.


### 订阅后的取消订阅方法

```javascript
subscription.unsubscribe();
```

### 订阅后的订阅方法

可以在取消订阅后恢复订阅:

```javascript
subscription.subscribe();
```

### 订阅后ready方法

一般只适用于同时订阅同1个通道但不同的逻辑的情况下:

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message;
});

// artificially model subscription to the same channel that happen after
// first subscription successfully subscribed - subscribe on the same
// channel after 5 seconds.
setTimeout(function() {
    var anotherSubscription = centrifuge.subscribe("news", function(message) {
        // another listener of channel "news"
    }).on("subscribe", function() {
        // won't be called on first subscribe because subscription already subscribed!
        // but will be called every time automatic resubscribe after network disconnect
        // happens
    });
    // one of subscribeSuccessHandler (or subscribeErrorHandler) will be called
    // only if subscription already subscribed (or subscribe request already failed).
    anotherSubscription.ready(subscribeSuccessHandler, subscribeErrorHandler);
}, 5000);
```

### 批量消息

允许一次性发送多条消息到服务器，特别是在通过SockJS拉取方式时特别有用:

```javascript
centrifuge.startBatching();
```

当你确实要发送批量消息时要调用`flush()` 方法:

```javascript
centrifuge.flush();
```

最后调用完毕后要进行关闭:

```javascript
centrifuge.stopBatching();
```

调用 `stopBatching(true)`可以清除所有消息并停止批量发送:

```javascript
centrifuge.stopBatching(true);
```


## 私有通道

如果通道以`$`开始则表示这是1个私有通道，这样的通道可以象下面这样订阅:

```javascript
centrifuge.subscribe('$private', function(message) {
    // process message
});
```

这样的订阅将会通过POST请求到你的后台 `/centrifuge/auth/`(可以通过变更Centrifugo设置来变更`authEndpoint`)进行验证. 该请求包括了`client`参数（客户端ID和`channels`参数） ，服务器进行验证后返回相应的内容。另外还有2个公共的方法可以批量订阅多个私有通道: `startAuthBatching`
和 `stopAuthBatching`. 当调用`startAuthBatching`后，js客户端会进行私有通道批量订阅，除非`stopAuthBatching()`被调用.

关于私有通道更多请 [查看相关章节](../mixed/private_channels.md).


## 连接检测

Javascript客户端支持刷新连接，它是依据Centrifugo中设置的`connection_lifetime`参数值，具体可以 [查看相关章节](../server/connection_check.md).
