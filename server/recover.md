# 恢复机制是如何工作的

`recover`参数从v1.2.0版本开始提供

这个功能选项受Last-Event-ID机制 [来自Eventsource协议](http://www.w3.org/TR/2012/WD-eventsource-20120426/)启发.

先说一下动机，我们生活在一个非理想的世界，有时网络会断开。当客户端离线的时候，有些消息可能发到通道中，但在客户端上线时这些消息有可能丢失了。Centrifugo可以自动进行恢复。

当然也存在Centrifugo不能恢复的情况，比如客户端断开超过了一周，而期间有大量的消息进入...这些场景下客户端只能依赖应用自身的实现来恢复。

只有客户端离线比较短的时间的情形下，Centrifugo可以自动在客户端重连后恢复丢失的消息。出于这个上的，客户端提供上次的消息ID，Centrifugo将会自动从消息历史中恢复。

恢复参数必须要配合`history_size` 和 `history_lifetime`一起使用。`history_size` 和 `history_lifetime`必须进行合理配置，同时通道要启用恢复功能。

注意，有时你的客户端可能不需要恢复。这些取决于你的发布机制和展现目的，并不是所有场景都需要恢复。

当然，如果你的应用自身实现了恢复机制，也可以手工从Centrifugo提取或是从你的应用后端中。

下面将说明如果在js客户端和Centrifugo服务器之间如何实现恢复。如果你只是使用，可以跳过下面的内容到下一章节。所有逻辑都在centrifuge-js客户端中.

客户端首次订阅通道时是不需要恢复的，所以发送订阅命令时可以象下面这样，避免Centrifugo去搜索历史消息中是否有需要恢复的消息：

```
{
    "channel": "news",
    "recover": false
}
```

Centrifugo收到客户端订阅的事件时反馈该通道上次的最后消息id：

```
{
    "last": "last-message-id-for-channel"
}
```

客户端得到这个ID后，每次收到一条消息，都会把该消息的id来覆盖，这样当客户端断开的时候可以通过这个id来恢复丢失的消息。

```
{
    "channel": "news",
    "recover": true,
    "last": "last-message-id-for-channel"
}
```

Centrifugo收到恢复的请求后就去消息历史中查找所有能恢复的消息然后返回给客户端:

```
{
    "messages": [message, message...]
}
```

这样客户端就可以处理这些消息并继续收取该通道的新消息。
