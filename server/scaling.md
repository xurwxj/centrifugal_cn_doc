基于Redis集群
==================

如你前面已经读到的 - 可以很方便的把Centrifugo的多个节点集群并负载均衡。这个章节我们将介绍如何实现它。我们将启动3个节点的Centrifugo通过Redis实现集群。

要实现这个目的我们必须要使用Redis引擎。

首先，你必须要运行Redis，然后我们打开终端并启动第1个Centrifugo节点:

```
centrifugo --config=config.json --port=8000 --engine=redis --redis_host=127.0.0.1 --redis_port=6379
```

如果你的Redis是运行在同一台服务器的默认端口上， 你可以省略`--redis_host`和`--redis_port`这2项配置。

启动第2个Centrifugo节点:

```
centrifugo --config=config.json --port=8001 --engine=redis --redis_host=127.0.0.1 --redis_port=6379
```

注意，由于我们是在同一台服务器上运行集群，所以，需要把第2个Centrifugo节点运行在不同的端口。如果你是在不同服务器上运行Centrifugo节点，可以不需要指定Centrifugo运行的端口。

启动第3个Centrifugo节点:

```
centrifugo --config=config.json --port=8002 --engine=redis --redis_host=127.0.0.1 --redis_port=6379
```

现在你拥有3个Centrifugo节点，分别运行在8000, 8001, 8002这3个端口，客户端已经可以连接他们了。你可以发送API请求到任一节点，Centrifugo所有节点会根据Redis的PUB/SUB消息来发送他们。

如果你使用Nginx来进行负载均衡，你可以在文档中找到它的配置。注意，当使用SockJS(排除websocket)时，要确保连接相同的节点（这个节点保存了客户端的连接会话信息）。

## Redis集群

从Centrifugo v1.6.0开始内建支持Redis集群支持。

在某些大型应用场景，这个可以解决Redis成为Centrifugo瓶颈的问题。Redis是单线程服务，它非常快，但如果CPU资源达到100%的时候，就需要集群来扩展了。这个特性支持你扩展你的应用支撑能力。

目前Centrifugo支持以逗号分隔的Redis集群，看下面的例子。

启动2个Redis集群支持的Centrifugo服务，Redis分别运行在6379和6380端口:

```
centrifugo --config=config.json --engine=redis --redis_port=6379,6380
```

启动运行在不同服务器上的Redis，并且用在Centrifugo上:

```
centrifugo --config=config.json --engine=redis --redis_host=192.168.1.34,192.168.1.35
```

如果你需要指定Redis的认证授权密码、指定Redis数据库名，你可以使用`--redis_url`参数。

注意，Redis集群的PUB/SUB不支持不同的数据库名。

启用了Redis集群（使用consistent hashing算法）的Centrifugo可以扩展通道、历史消息、在线状态等。目前我们使用Jump consistent hash算法
(看 [论文](https://arxiv.org/pdf/1406.2294.pdf) and [实现](https://github.com/dgryski/go-jump))

如果你有任何关于Redis集群特性的反馈，请及时告知我们。
