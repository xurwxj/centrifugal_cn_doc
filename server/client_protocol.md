# 客户端协议描述

这个章节的目的是帮助开发者实现新的客户端库或者理解已有的客户端库是怎么实现的。这个章节并不完事，当有需要时会进行更新。

Centrifugo已经有基于Javascript, Go, iOS, Android, Python的库可以用在你的应用中。

了解如何实现新客户端库的一个方式是查看已有库的源代码，比如[centrifuge-js](https://github.com/centrifugal/centrifuge-js/blob/master/src/centrifuge.js) 或 [centrifuge-python](https://github.com/centrifugal/centrifuge-python).

当前的websocket是唯一实现客户端的可用方式。Centrifugo也同时支持在浏览器中使用SockJS。所有通信通过JSON格式的消息进行交换。

下面我们一步步了解客户端协议。

### 连接、订阅到通道并且等待发布的消息

Websocket地址是:

```
ws://your_centrifugo_server.com/connection/websocket
```

或者是使用TLS:

```
wss://your_centrifugo_server.com/connection/websocket
```

首先客户端需要创建一个Websocket连接，连接成功后，客户端必须要发送`connect`到Centrifugo服务器来认证自己。

`connect`命令是一个JSON结构的对象，下面使用Javascript语言来做样例（其它语言是一样的结构）:

```javascript
var message = {
    "uid": "UNIQUE COMMAND ID",
    "method": "connect",
    "params": {
        "user": "USER ID STRING",
        "timestamp": "STRING WITH CURRENT TIMESTAMP SECONDS",
        "info": "OPTIONAL JSON ENCODED STRING",
        "token": "SHA-256 HMAC TOKEN GENERATED FROM PARAMETERS ABOVE"
    }
}

connection.send(JSON.stringify(message))
```

注意`method`属性值就是我们的命令：`connect`.

Centrifugo可以在一次请求中解析一个消息数组，所以你可以把上述命令加到数组中通过Websocket连接发送:

```javascript
var messages = [message]
connection.send(JSON.stringify(messages))
```

`connect`有以下参数可以使用:

* `user` - 当前的用户ID (string)
* `timestamp` - 当前以秒为单位的UNIX时间戳(string)
* `info` - 可选的客户端其它信息 (string)
* `token` - 根据前面章节描述生成的SHA-256 HMAC token.

客户端必须要提供这些连接参数，Centrifugo在接到命令后会使用同样的算法生成token。

*注意* Centrifugo也允许匿名接入，请查看前面的配置章节。

下面要做的就是等等Centrifugo返回的响应结果，一般来说，Centrifugo返回的内容结构如下:

```javascript
[{response}, {response}, {response}]
```

或者只是单个响应体

```
{response}
```

是数组还是单个响应体是根据你之前发送的命令数量对应的。相对上面的代码示例来说，Centrifugo将返回:

```javascript
[{connect_command_response}]
```

或者仅是:

```javascript
{connect_command_response}
```

**客户端必须要自行处理数据或单个响应体，所有命令都是同样的返回方式**.

每个`response`结构如下:

```javascript
{
    "uid": "ECHO BACK THE SAME UNIQUE COMMAND ID SENT IN REQUEST COMMAND",
    "method": "COMMAND NAME TO WHICH THIS RESPONSE REFERS TO",
    "error": "ERROR STRING, IF NOT EMPTY THEN SOMETHING WENT WRONG AND BODY SHOULD NOT BE PROCESSED",
    "body": "RESPONSE BODY, CONTAINS USEFUL RESPONSE DATA"
}
```

Javascript客户端使用`method`属性来了解响应的结果对应。因为Javascript是事件驱动性语言，它只需要调用相应的方法来处理响应即可。唯一的`uid`也可以用于其它语言的实现处理。比如Golang客户端就在收到Centrifugo的响应时通过`uid`来实现一些回调。

一般约定：如果响应体中有一个非空的`error`则表示Centrifugo返回了一个错误提示。

在正常情况下，你不会拿到错误。如果你拿到了错误，一般是因为你做错了一些事，这些需要在开发阶段进行修复。当然也有可能是Centrifugo的`internal server error`。你应该确保这些错误信息只有开发者能看到，而不是你的用户。

如果是成功的`connect`，它的响应内容类似如下:

```javascript
{
    "client": "UNIQUE CLIENT ID SERVER GAVE TO THIS CONNECTION",
    "expires": "false",
    "expired": false,
    "ttl": 0
}
```

这里要说一下`client`属性，这是Centrifugo为这个连接设置的唯一客户端ID。

一旦你的客户端连接成功并拿到了唯一的客户端ID，你就可以订阅通道了。

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "CHANNEL TO SUBSCRIBE"
    }
}
```

只需与`connect`命令同样的方式发送`subscribe`命令即可。当你得到成功的响应后，你就可以收到这个通道中被发布的消息了。并且这些消息后续会通过Websocket连接通过命令`message`发送过来，这些消息的格式如下:

```
{
    "method": "message",
    "body": {
        "uid": "8d1f6279-2d13-45e2-542d-fac0e0f1f6e0",
        "info":{
            "user":"42",
            "client":"73cd5abb-03ed-40bc-5c87-ed35df732682",
            "default_info":null,
            "channel_info":null
        },
        "channel":"jsfiddle-chat",
        "data": {
            "input":"hello world"
        },
        "client":"73cd5abb-03ed-40bc-5c87-ed35df732682"
    }
}
```

`message`命令响应的`body`属性值包含了消息所属的`channel`和在`data`中的实际内容。

到这里，基本的连接、认证、订阅都已经讲完了，涵盖了Centrifugo的核心功能。还有其它可以讲的，包括通道在线信息、通道历史信息、连接过期、私有通道订阅、加入/离开事件等.

### 可用的命令

下面是你的客户端可以发送或接收的所有可用命令:

```
connect
disconnect
subscribe
unsubscribe
publish
presence
history
join
leave
message
refresh
ping
```

部分命令是客户端发到服务器的，比如`publish`, `presence`, `history`等，这些命令从服务器返回同样的格式：`method`和唯一的 `uid`；部分是服务器发到客户端的，比如`join`, `leave`, `message`，这些只在事件发生的时候才会从服务器发出.

我们已经讲了`connect`, `subscribe` 和 `publish`，接下来讲其它的.

### 客户端发到服务器的命令

`connect` - 发送认证到Centrifugo以便于后续操作.

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'connect',
    'params': {
        'user': "USER ID STRING",
        'timestamp': "STRING WITH CURRENT TIMESTAMP SECONDS"
        'info': "OPTIONAL JSON ENCODED STRING",
        'token': "SHA-256 HMAC TOKEN GENERATED FROM PARAMETERS ABOVE"
    }
}
```

`subscribe` - 连接成功允许订阅通道

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "CHANNEL TO SUBSCRIBE"
    }
}
```

`unsubscribe` - 允许取消订阅通道

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "unsubscribe",
    "params": {
        "channel": "CHANNEL TO UNSUBSCRIBE"
    }
}
```

`publish` - 允许客户端直接发布消息到通道（注意后端应用不会知道这类消息），要使用`publish`要在配置中先启用，否则Centrifugo会返回`permission denied`错误.

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "publish",
    "params": {
        "channel": "CHANNEL",
        "data": {}  // JSON DATA TO PUBLISH
    }
}
```

`presence` – 允许从服务器端拿到通道在线信息，要使用`presence`，必须要对通道配置进行启用，否则Centrifugo会返回`not available`错误

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "presence",
    "params": {
        "channel": "CHANNEL"
    }
}
```

`history` – 允许从服务器拿到通道历史信息，要先在通道配置中启用`history_lifetime` 和 `history_size`选项，否则Centrifugo会返回`not available`错误

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "history",
    "params": {
        "channel": "CHANNEL"
    }
}
```

`ping` - 允许发送ping命令到服务器，服务器则同样返回`ping`的响应

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "ping"
}
```

### 服务器发到客户端的命令

`message` - 新消息被发到当前客户端订阅的通道，格式如下:

```javascript
{
    "method":"message",
    "body": {
        "uid": "8d1f6279-2d13-45e2-542d-fac0e0f1f6e0",
        "info":{
            "user":"42",
            "client":"73cd5abb-03ed-40bc-5c87-ed35df732682",
            "default_info":null,
            "channel_info":null
        },
        "channel":"jsfiddle-chat",
        "data": {
            "input":"hello world"
        },
        "client":"73cd5abb-03ed-40bc-5c87-ed35df732682"
    }
}
```

`join` - 有新的客户端同样订阅了当前客户端订阅的通道时发生。注意`join_leave`选项在通道配置中必须要启用才能收到这类消息。`body`属性包含了新客户端的信息：

```javascript
{
    "method":"join",
    "body": {
        "channel":"$public:chat",
        "data": {
            "user":"2694",
            "client":"3702659c-f28a-4166-5b44-115d9b544b29",
            "default_info": {
                "first_name":"Alexandr",
                "last_name":"Emelin"
            },
            "channel_info": {
                "channel_extra_info_example":"you can add additional JSON data when authorizing"
            }
        }
    }
}
```

`leave` - 有客户端取消订阅当前客户端订阅的通道时发生。注意`join_leave`选项在通道配置中必须要启用才能收到这类消息。`body`属性包含了取消订阅的客户端的信息：

```javascript
{
    "method":"leave",
    "body": {
        "channel":"$public:chat",
        "data": {
            "user":"2694",
            "client":"3702659c-f28a-4166-5b44-115d9b544b29",
            "default_info": {
                "first_name":"Alexandr",
                "last_name":"Emelin"
            },
            "channel_info": {
                "channel_extra_info_example":"you can add additional JSON data when authorizing"
            }
        }
    }
}
```

### 私有通道订阅

注意连接成功返回的`client`属性，这是一个Centrifugo设定的唯一连接ID，用于认证获取私有通道签名。

上面已经讲了订阅普通的通道可以用以下的格式命令:

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "CHANNEL TO SUBSCRIBE"
    }
}
```

当订阅私有通道时，必须还要额外提供一些信息在`params`属性对象中:

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "channel to subscribe",
        'client': "current client ID",
        'info': "additional private channel JSON string info",
        'sign': "string channel sign generated on app backend based on client ID and optional info"
    }
}
```

查看 [关于签名的章节](./tokens_and_signatures.md)来了解更多如何认证的信息。

客户端库至少要提供一个机制能提到当前连接的客户端ID来设置`params`需要的`client`, `info` 和 `sign`值，注意每次重新连接都会发生变化，所以每次连接前要务必先获取签名。

