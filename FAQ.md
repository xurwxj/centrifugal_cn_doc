# 常见问题


### 单个Centrifugo实例可以支持多少连接?

这个要根据实际情况而定，硬件、消息率、消息大小、通道的设置、客户端的分发以及websocket压缩都会影响连接数，一般来说，我们建议一个节点不超过50-100k个客户端，但你可以自行估量.

### Centrifugo支持横向扩展吗?

可以，基于Redis雅苑，对于大多数情况来说是可以的，但你要考虑在不同Redis实例或集群中同步、负载均衡.

### 为什么Centrifugo阻止新的接入?

大部分原因是达到文件打开上限，请参照系统优化部分进行优化或是进行硬件横向扩展.

### 我可以不用Nginx之类的代理服务器来跑Centrifugo吗?

可以，但代理服务器对于负载均衡支持更大连接数来说有非常大的什么用.

### Centrifugo支持HTTP/2吗?

支持，你也可以在启动时传递`GODEBUG`环境变量来禁用:

```
GODEBUG="http2server=0" centrifugo -c config.json
```

### 同1浏览器的多窗口如何共用同1个连接?

如果使用HTTP/2是自动共用的，如果是websocket连接，这里可以使用`SharedWorker`对象来实现.

### 如果我要推送到mobile或web应用需要做什么?

实际上，对于实时消息和推送提醒有时是比较混淆的，Centrifugo是个实时消息服务器，它不推送提醒到设备，如果你要通过Centrifugo来实现，可通过数据库来实现ack，然后定时去检测、重发.

### 我怎么知道消息真的被发到客户端了?

你自己可以实现但Centrifugo没有这样的API，你可以让客户端发一个ack确认来保证.

### 我可以在客户端通过websocket连接发布新消息吗?

支持的。

### 如何为2个用户创建安全通道（私聊模式）?

有好几种方式:

* 使用私有通道(以`$`开头命名) - 具体 [查看这里](https://fzambia.gitbooks.io/centrifugal/content/mixed/private_channels.html)
* 使用[用户限制的通道](https://fzambia.gitbooks.io/centrifugal/content/server/channels.html#user-channel-boundary) (以`#`开头) 
* 你可以创建不容易猜的通道名称来实现.

### 组织通道配置的最好方式是什么?

使用命名空间来批量设置共用的参数.

### 我能依赖Centrifugo及它的历史消息来保证传递吗?

不能，Centrifugo只能在有限情况下可以保证传递，要完整实现请自行设计.

### 没找到答案的请到[gitter聊天室](https://gitter.im/centrifugal/centrifugo)
