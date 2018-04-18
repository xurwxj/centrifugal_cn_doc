# 引擎

* [内存引擎](#memory-engine)
* [Redis引擎](#redis-engine)

Centrifugo中的引擎是用来发布消息、处理订阅、保存/取回在线状态和历史数据。

Centrifugo默认使用内存引擎。另外也可以配置使用Redis引擎。

2个引擎最明显的区别是：使用内存引擎你不能做集群，而Redis引擎可以。

你可以使用`engine`选项配置引擎，值可以是`memory` 或 `redis`，默认值是 `memory`.

也可以在命令行中指定使用Redis引擎:

```
centrifugo --config=config.json --engine=redis
```

### 内存引擎

仅支持单机，简单易用，无须安装Redis，在内存中处理所有特性。


### Redis引擎

允许在不同的机器上进行集群，节点间使用Redis作为消息中介。Redis引擎在Redis中保存在线状态和历史数据，并且在节点间使用Redis进行发布和订阅通信，它也支持api命令队列。

![架构](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/scheme_redis.png)

如何通过Redis引擎的API监听发布？启动使用Redis引擎的Centrifugo并配置``--redis_api`` 选项:

```bash
centrifugo --config=config.json --engine=redis --redis_api
```

注意，v1.6.0版本之后，Centrifugo的Redis API 消息格式改变了，你可以在[这里](https://github.com/centrifugal/documentation/blob/2eadd7d3f9991c46c463aff4126f2ea37b17bfad/server/engines.md#redis-engine)找到说明，旧格式将在未来的版本中被移除。

使用你喜爱的语言来接入Redis，比如Python:

```python
import redis
import json

client = redis.Redis()

command = {
    "method": "publish",
    "params": {
        "channel": "events",
        "data": {"event": "message"}
    }
}

client.rpush("centrifugo.api", json.dumps(command))
```

[RPUSH](https://redis.io/commands/rpush) Redis命令允许在一个请求中推送多个消息到队列中。

上面的代码中，我们RPUSH 消息到`centrifugo.api`队列 - 这是A监听的默认API队列名称，其实只是Redis中一个列表结构的数据。

另外，添加到队列后你也不需要响应，这些队列中的消息会被Centrifugo在尽可能短的时间处理掉，如果你需要响应，你应该使用HTTP API。

`publish`是最有用的API命令，Redis API监听器实现的最主要目的是为了减少HTTP压力，同时也有助于那些尚未有HTTP API客户端的语言能使用。

几个与Redis引擎相关的配置选项说明:

Run

```bash
centrifugo -h
```

你将看到其中以下的一些参数:

```bash
    --redis_api                  启用Redis API监听器 (Redis engine)
    --redis_api_num_shards int   redis API队列的共享池大小 (Redis engine)
    --redis_db string            redis数据库名 (Redis engine) (默认是 "0")
    --redis_host string          redis服务器地址 (Redis engine) (默认是 "127.0.0.1")
    --redis_master_name string   Redis master Sentinel监控名称 (Redis engine)
    --redis_password string      redis用户密码(Redis engine)
    --redis_pool int             Redis连接池大小 (Redis engine) (默认是 256)
    --redis_port string          redis端口 (Redis engine) (默认是 "6379")
    --redis_sentinels string     逗号分隔的Sentinels列表 (Redis engine)
    --redis_url string           redis连接URL，格式为： redis://:password@hostname:port/db (Redis engine)
```

这些参数大部分是很简洁明了的 – `--redis_host`, `--redis_port`, `--redis_password`, `--redis_db`, `--redis_pool`

`--redis_url` 允许通过一定格式的URL来连接Redis，格式为：`redis://:password@hostname:port/db_number`.

当 `--redis_url`设置时，Centrifugo会忽略`--redis_host`,`--redis_port`, `--redis_password`, `--redis_db` 这些参数的值.

最高级的参数是`--redis_api_num_shards`. 它是v1.3.0开始新加的，这个参数必须结合`--redis_api`, 因为只有启用了Redis API，这个参数才有效。它创建N个额外的共享队列，Centrifugo用这些队列来监听新的API命令.

如果你设置`--redis_api_num_shards`为5，则Centrifugo在Redis中监听以下队列:

```
centrifugo.api.0
centrifugo.api.1
centrifugo.api.2
centrifugo.api.3
centrifugo.api.4
```

你可以发布新的命令到任一上述队列中，命令会被Centrifugo实例接收并处理。为什么我们需要这个？

就象上面我们启用`--redis_api`通过RPUSH发布新消息到Redis的`centrifugo.api`队列，这种做法在少量消息时没问题，但当每秒成千上万的消息要发布时，解决方案就是上面的共享队列。要注意的是，你必须在客户端自己指定要发布到哪个队列中。要保持通道中的消息顺序，你必须要确保属于同一通道的消息发布到同样的队列中。这个可以使用象`crc16(CHANNEL_NAME) mod N`这样的方法进行归档，这里N是共享队列的数量(i.e. ``--redis_api_num_shards``设置的值).

下一章节我们将看到如何使用Redis引擎启动多个Centrifugo节点.
