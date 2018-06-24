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

You can optionally provide extra parameter `info` when connecting to Centrifugo, i.e.:

```javascript
var centrifuge = new Centrifuge({
    url: 'http://centrifuge.example.com/connection',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    info: '{"first_name": "Alexandr", "last_name": "Emelin"}',
    token: 'TOKEN'
});
```

`info` is an additional information about connection. It must be **valid encoded JSON string**.
But to prevent client sending wrong `info` **this JSON string must be used while generating
token**.

If you don't want to use `info` - you can just omit this parameter while connecting to Centrifugo.
But if you omit it then make sure that info string have not been used in token generation
(i.e. `info` must be empty string).

## Configuration parameters

Let's also look at optional configuration parameters available when initializing
`Centrifuge` object instance.

#### transports

In case of using SockJS additional configuration parameter can be used - `transports`.

It defines allowed SockJS transports and by default equals

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

i.e. all possible SockJS transports.

So to say `centrifuge-js` to use only `websocket` and `xhr-streaming` transports when
using SockJS endpoint:

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

**new in 1.3.7**. `sockJS` option allows to explicitly provide SockJS client object to Centrifuge client.

For example this can be useful if you develop in ES6 using imports.

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

`debug` is a boolean option which is `false` by default. When enabled lots of various debug
messages will be logged into javascript console. Mostly useful for development or
troubleshooting.

#### insecure

`insecure` is a boolean option which is `false` by default. When enabled client will connect
to server in insecure mode - read about this mode in [special docs chapter](../mixed/insecure_modes.md).

This option nice if you want to use Centrifugo for quick real-time ideas prototyping, demos as
it allows to connect to Centrifugo without `token`, `timestamp` and `user`. And moreover without
application backend! Please, [read separate chapter about insecure modes](../mixed/insecure_modes.md).

#### retry

When client disconnected from server it will automatically try to reconnect using exponential
backoff algorithm to get interval between reconnect attempts which value grows exponentially.
`retry` option sets minimal interval value in milliseconds. Default is `1000` milliseconds.

#### maxRetry

`maxRetry` sets upper interval value limit when reconnecting. Or your clients will never reconnect
as exponent grows very fast:) Default is `20000` milliseconds.

#### resubscribe

`resubscribe` is boolean option that allows to disable automatic resubscribing on
subscriptions. By default it's `true` - i.e. you don't need to manually handle
subscriptions resubscribing and no need to wait `connect` event triggered (first
time or when reconnecting) to start subscribing. `centrifuge-js` will by default
resubscribe automatically when connection established.

#### server

`server` is SockJS specific option to set server name into connection urls instead
of random chars. See SockJS docs for more info.

#### authEndpoint

`authEndpoint` is url to use when sending auth request for authorizing subscription
on private channel. By default `/centrifuge/auth/`. See also useful related options:

* `authHeaders` - map of headers to send with auth request (default `{}``)
* `authParams` - map of params to include in auth url (default `{}`)
* `authTransport` - transport to use for auth request (default `ajax`, possible value `jsonp`)

#### refreshEndpoint

`refreshEndpoint` is url to use when refreshing client connection parameters when
connection check mechanism enabled in Centrifugo configuration. See also related
options:

* `refreshHeaders` - map of headers to send with refresh request (default `{}``)
* `refreshParams` - map of params to include in refresh url (default `{}`)
* `refreshTransport` - transport to use for refresh request (default `ajax`, possible value `jsonp`)
* `refreshData` - send extra data in body (as JSON payload) when sending AJAX POST refresh request.
* `refreshAttempts` - limit amount of refresh requests before giving up (by default `null` - unlimited)
* `refreshFailed` - callback function called when `refreshAttempts` came to the end. By default `null` - i.e. nothing called.

## Client API

When `Centrifuge` object properly initialized then it is ready to start communicating
with server.

#### connect method

As we showed before, we must call `connect()` method to make an actual connection
request to Centrifugo server:

```javascript
var centrifuge = new Centrifuge({
    // ...
});

centrifuge.connect();
```

`connect()` calls actual connection request to server with connection parameters and
configuration options you provided during initialization.

#### connect event

After connection will be established and client credentials you provided authorized
then `connect` event on `Centrifuge` object instance will be called.

You can listen to this setting event listener function on `connect` event:

```javascript
centrifuge.on('connect', function(context) {
    // now client connected to Centrifugo and authorized
});
```

What's in `context`:

```javascript
{
    client: "79ec54fa-8348-4671-650b-d299c193a8a3",
    transport: "raw-websocket",
    latency: 21
}
```

* `client` – client ID Centrifugo gave to this connection (string)
* `transport` – name of transport used to establish connection with server (string)
* `latency` – latency in milliseconds (int). This measures time passed between sending
    `connect` client protocol command and receiving connect response. New in 1.3.1

#### disconnect event

`disconnect` event fired on centrifuge object every time client disconnects for
some reason. This can be network disconnect or disconnect initiated by Centrifugo server.

```javascript
centrifuge.on('disconnect', function(context) {
    // do whatever you need in case of disconnect from server
});
```

What's in `context`?

```javascript
{
    reason: "connection closed",
    reconnect: true
}
```

* `reason` – the reason of client's disconnect (string)
* `reconnect` – flag indicating if client will reconnect or not (boolean)


#### error event

`error` event called every time on centrifuge object when response with error received.
In normal workflow it will never be happen. But it's better to log these errors to detect
where problem with connection is.

This event handler is a general error messages sink - it will receives all messages received
from Centrifugo containing error so it could also receive message resulting in `error` event
for subscription (see below). The difference is that this event handler exists mostly for
logging purposes to help developer fix possible problems - while other errors (subscription
error or `publish`, `presence`, `history` call errors) can be theoretically handled to retry
call or resubscribe maybe.

```javascript
centrifuge.on('error', function(error) {
    // handle error in a way you want, here we just log it into browser console.
    console.log(error)
});
```

What's in `error`?

```javascript
{
    "message": {
        "method": "METHOD",
        "error": "ERROR DESCRIPTION",
        "advice": "OPTIONAL ERROR ADVICE"
    }
}
```

`message` – message from server containing error. It's a raw protocol message resulted in
error event because it contains `error` field. At bare minimum it's recommended to log these
errors. In normal workflow such errors should never exist and must be a signal for developer
that something goes wrong.


#### disconnect method

In some cases you may need to disconnect your client from server, use `disconnect` method to
do this:

```javascript
centrifuge.disconnect();
```

After calling this client will not try to reestablish connection periodically. You must call
`connect` method manually again.


## Subscriptions

Of course being just connected is useless. What we usually want from Centrifugo is to
receive new messages published into channels. So our next step is `subscribe` on channel
from which we want to receive real-time messages.


### subscribe method

To subscribe on channel we must use `subscribe` method of `Centrifuge` object instance.

The simplest usage that allow to subscribe on channel and listen to new messages is:

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle new message coming from channel "news"
    console.log(message);
});
```

And that's all! For lots of cases it's enough! But let's look at possible events that
can happen with subscription:

* `message` – called when new message received (callback function in our previous example is `message`
    event callback btw)
* `join` – called when someone joined channel
* `leave` – called when someone left channel
* `subscribe` – called when subscription on channel successful and acknowledged by Centrifugo
    server. It can be called several times during javascript code lifetime as browser client
    automatically resubscribes on channels after successful reconnect (caused by temporary
    network disconnect for example or Centrifugo server restart).
* `error` – called when subscription on channel failed with error. It can called several times
    during javascript code lifetime as browser client automatically resubscribes on channels
    after successful reconnect (caused by temporary network disconnect for example or Centrifugo
    server restart).
* `unsubscribe` – called every time subscription that was successfully subscribed
    unsubscribes from channel (can be caused by network disconnect or by calling
    `unsubscribe` method of subscription object)

Don't be frightened by amount of events available. In most cases you only need some of them
until you need full control to what happens with your subscriptions. We will look at format
of messages for this event callbacks later below.

There are 2 ways setting callback functions for events above.

First is providing object containing event callbacks as second argument to `subscribe` method.

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

Another way is setting callbacks using `on` method of subscription. Subscription object
is event emitter so you can simply do the following:

```javascript
var subscription = centrifuge.subscribe("news");

subscription.on("message", messageHandlerFunction);
subscription.on("subscribe", subscribeHandlerFunction);
subscription.on("error", subscribeErrorHandlerFunction);
```

***Subscription objects are instances of [EventEmitter](https://github.com/Olical/EventEmitter/blob/master/docs/api.md).***


### join and leave events of subscription

As you know you can enable `join_leave` option for channel in Centrifugo configuration.
This gives you an opportunity to listen to `join` and `leave` events in those channels.
Just set event handlers on `join` and `leave` events of subscription.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
}).on("join", function(message) {
    console.log("Client joined channel");
}).on("leave", function(message) {
    console.log("Client left channel");
});
```


### subscription event context formats

We already know how to listen for events on subscription. Let's look at format of
messages event callback functions receive as arguments.

#### format of message event context

Let's look at message format of new message received from channel:

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "channel":"$public:chat",
    "data":{"input":"hello"},
}
```

I.e. `data` field contains actual data that was published.

Message can optionally contain `client` field (client ID that published message) - if
it was provided when publishing new message:

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "channel":"$public:chat",
    "data":{"input":"hello"},
    "client":"7080fd2a-bd69-4f1f-6648-5f3ceba4b643"
}
```

And it can optionally contain additional client `info` in case when this message was
published by javascript client directly using `publish` method (see details below):

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


#### format of join/leave event message

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

`default_info` and `channel_info` exist in message only if not empty.


#### format of subscribe event context

I.e. `on("subscribe", function(context) {...})`

```javascript
{
    "channel": "$public:chat",
    "isResubscribe": true
}
```

`isResubscribe` – flag showing if this was initial subscribe (`false`) or resubscribe (`true`)


#### format of subscription error event context

I.e. `on("error", function(err) {...})`

```javascript
{
    "error": "permission denied",
    "advice": "fix",
    "channel": "$public:chat",
    "isResubscribe": true
}
```

`error` - error description
`advice` - optional advice (`retry` or `fix` at moment)
`isResubscribe` – flag showing if this was initial subscribe (`false`) or resubscribe (`true`)


#### format of unsubscribe event context

I.e `on("unsubscribe", function(context) {...})`

```
{
    "channel": "$public:chat"
}
```

I.e. it contains only `channel` at moment.


### presence method of subscription

`presence` allows to get information about clients which are subscribed on channel at
this moment. Note that this information is only available if `presence` option enabled
in Centrifugo configuration for all channels or for channel namespace.

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

`presence` is internally a promise that will be resolved with data or error only
when subscription actually subscribed.

Format of success callback `message`:

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

As you can see presence data is a map where keys are client IDs and values are objects
with client information.

Format of `err` in error callback:

```javascript
{
    "error": "timeout",
    "advice": "retry"
}
```

* `error` – error description (string)
* `advice` – error advice (string, "fix" or "retry" at moment)


### history method of subscription

`history` method allows to get last messages published into channel. Note that history
for channel must be configured in Centrifugo to be available for `history` calls from
client.

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

Success callback `message` format:

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

Where `data` is an array of messages published into channel.

Note that also additional fields can be included in messages - `client`, `info` if those
fields were in original messages.

`err` format – the same as for `presence` method.


### publish method of subscription

`publish` method of subscription object allows to publish data into channel directly
from client. The main idea of Centrifugo is server side only push. Usually your application
backend receives new event (for example new comment created, someone clicked like button
etc) and then backend posts that event into Centrifugo over API. But in some cases you may
need to allow clients to publish data into channels themselves. This can be used for demo
projects, when prototyping ideas for example, for personal usage. And this allow to make
something with real-time features without any application backend at all. Just javascript
code and Centrifugo.

**So to emphasize: using client publish is not an idiomatic Centrifugo usage. It's not for
production applications but in some cases (demos, personal usage, Centrifugo as backend
microservice) can be justified and convenient. In most real-life apps you need to send new
data to your application backend first (using the convenient way, for example AJAX request
in web app) and then publish data to Centrifugo over Centrifugo API.**

To do this you can use `publish` method. Note that just like presence and history publish
must be allowed in Centrifugo configuration for all channels or for channel namespace. When
using `publish` data will go through Centrifugo to all clients in channel. Your application
backend won't receive this message.

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

`err` format – the same as for `presence` method.


### unsubscribe method of subscription

You can call `unsubscribe` method to unsubscribe from subscription:

```javascript
subscription.unsubscribe();
```

### subscribe method of subscription

You can restore subscription after unsubscribing calling `.subscribe()` method:

```javascript
subscription.subscribe();
```

### ready method of subscription

A small drawback of setting event handlers on subscription using `on` method is that event
handlers can be set after `subscribe` event of underlying subscription already fired. This
is not a problem in general but can be actual if you use one subscription (i.e. subscription
to the same channel) from different parts of your javascript application - so be careful.

For this case one extra helper method `.ready(callback, errback)` exists. This method calls
`callback` if subscription already subscribed and calls `errback` if subscription already
failed to subscribe with some error (because you subscribed on this channel before). So
when you want to call subscribe on channel already subscribed before you may find `ready()`
method useful:

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

When called `callback` and `errback` of `ready` method receive the same arguments as
callback functions for `subscribe` and `error` events of subscription.


### Message batching

There is also message batching support. It allows to send several messages to server
in one request - this can be especially useful when connection established via one of
SockJS polling transports.

You can start collecting messages to send calling `startBatching()` method:

```javascript
centrifuge.startBatching();
```

When you want to actually send all collected messages to server call `flush()` method:

```javascript
centrifuge.flush();
```

Finally if you don't want batching anymore call `stopBatching()` method:

```javascript
centrifuge.stopBatching();
```

Call `stopBatching(true)` to flush all messages and stop batching:

```javascript
centrifuge.stopBatching(true);
```


## Private channels

If channel name starts with `$` then subscription on this channel will be checked via
AJAX POST request from javascript client to your web application backend.

You can subscribe on private channel as usual:

```javascript
centrifuge.subscribe('$private', function(message) {
    // process message
});
```

But in this case client will first check subscription via your backend sending POST request
to `/centrifuge/auth/` endpoint (by default, can be changed via configuration option
`authEndpoint`). This request will contain `client` parameter which is your connection
client ID and `channels` parameter - one or multiple private channels client wants to
subscribe to. Your server should validate all this subscriptions and return properly
signed responses.

There are also two public API methods which can help to subscribe to many private
channels sending only one POST request to your web application backend: `startAuthBatching`
and `stopAuthBatching`. When you `startAuthBatching` javascript client will collect
private subscriptions until `stopAuthBatching()` called – and then send them all at
once.

Read more about private channels in [special documentation chapter](../mixed/private_channels.md).


## Connection check

Javascript client has support for refreshing connection when `connection_lifetime` option
set in Centrifugo. See more details in [dedicated chapter](../server/connection_check.md).
