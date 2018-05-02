# 服务端API

除了HTTP API外，还有一种方式可以发送命令给Centrifugo – 就是使用Redis引擎，但这章节我们将主要介绍HTTP API.

为什么我们需要API?

如果你看到项目和命名空间参数，你会看到有一个参数叫`publish`。当启用这个参数的时候，就允许浏览器客户端直接发送信不信到这个通道。如果客户端直接发布消息到通道，你的应用将不会接收到这些消息（他们是直接通过Centrifugo转发到了订阅的客户端）。这种模式有时非常有用，但在大部分时候，你需要通过ajax接收、处理消息 - 可能用于验证、保存然后再通过HTTP API `publish`到Centrifugo，然后Centrifugo再广播给所有订阅的客户端。

HTTP API也可以用于发送其它命令 - 所有可用的命令请看下面。

如果你使用Python，你可以使用[Cent](../libraries/python.md) API客户端。我们也有[Ruby](../libraries/ruby.md)的客户端和[PHP](../libraries/php.md)的客户端。如果你使用其它语言，请不要担心，我会在这个章节说明如何接入Centrifugo HTTP API。

注意，Centrifugo有HTTP API和Redis引擎API这2种API接入方式，这章我们主要说HTTP API。你可以在`engines`章节查找Redis引擎API的接入方式 - 它有同样的命令，但格式更加简单。
所以如果你觉得HTTP API难用、不好用 - 你可以使用Redis引擎API来发布消息到通道。

现在开始介绍HTTP API!

Centrifugo API的url是`/api/`.

如果你的Centrifugo站点域名是`https://centrifuge.example.com`，那API地址就是`https://centrifuge.example.com/api/`。

All you need to do to use HTTP API is to send correctly constructed POST request to this endpoint.

API请求是`application/json`格式的POST请求，命令及内容在请求的body中，并且需要在请求头中设置`X-API-Sign`。命令可以是单个或数组形式的多个。`X-API-Sign`头是SHA-256 HMAC格式的字符串，基于Centrifugo加密密钥和你要发送的JSON内容，用于Centrifugo验证API请求合法性。

大部分情况下，你可以通过防火墙和`--insecure_api`参数来禁用验证，就足够保证Centrifugo安全性。如果`--insecure_api`被启用，则无需设置上述头信息。
命令内容是一个JSON对象，这个对象有2个属性: `method` 和 `params`.

`method` 是你想执行的行为.
`params` 是行为的参数，它是一个对象.

下面是通过Python的代码样例来展示如何构建JSON对象:

```
command = json.dumps({
    "method": "publish",
    "params": {"channel": "news", data:{}}
})
```

要发布2个命令，你可以象下面这样:

```
commands = json.dumps([
    {
        "method": "publish",
        "params": {"channel": "news", data:{"content": "1"}}
    },
    {
        "method": "publish",
        "params": {"channel": "news", data:{"content": "2"}}
    }
])
```

现在，我们来看看Python中是怎么构建这样的请求的，这对于想自行实现Centrifugo API或想自己发送请求（减少额外的项目依赖）的人非常有用。

假设你的Centrifugo加密密钥是`secret`.

```python
import json
import requests

from cent.core import generate_api_sign


commands = [
    {
        "method": "publish",
        "params": {"channel": "docs", "data": {"content": "1"}}
    }
]
encoded_data = json.dumps(commands)
sign = generate_api_sign("secret", encoded_data)
headers = {'Content-type': 'application/json', 'X-API-Sign': sign}
r = requests.post("https://centrifuge.example.com/api/", data=encoded_data, headers=headers)
print r.json()
```

上述代码生成了认证签名，如果你想自己生成API签名，你可以跳到章节[Tokens和签名](./tokens_and_signatures.md)。

同时，我们在上述代码中发送了命令数组，这意味着我们可以一次发送多个命令给Centrifugo。

你可以调用的命令主要是以下（其中最主要、最有用的是`publish`，还有`broadcast`, `unsubscribe`, `presence`, `history`, `disconnect`,
`channels`, `stats`, `node`）：

### publish

`publish`允许你发送消息到通道。`params`必须是一个有`channel`和`data`属性的对象，这2个属性决定了要发送到通道的内容。

```javascript
{
    "method": "publish",
    "params": {
        "channel": "CHANNEL NAME",
        "data": {
            "input": "hello"
        }
    }
}
```

从**v0.2.0**开始，增加了一个`client`的ID可选项:

```javascript
{
    "method": "publish",
    "params": {
        "channel": "CHANNEL NAME",
        "data": {
            "input": "hello"
        },
        "client": "long-unique-client-id"
    }
}
```

大部分情况下它就是用来初始化消息的`client` ID，`client`的值将添加到被发布消息的顶部。

响应样例如下:

```javascript
{
    "body": null,
    "error": null,
    "method": "publish"
}
```

### broadcast (从v1.2.0开始增加)

和`publish`很相似，但允许发布同样的数据到很多通道去。

```javascript
{
    "method": "broadcast",
    "params": {
        "channels": ["CHANNEL_1", "CHANNEL_2"],
        "data": {
            "input": "hello"
        }
    }
}
```

`client`也是用于设置客户端ID。 

该命令将持续发布数据至通道，除非发生错误才停止。如果使用Redis API队列，则第一个错误日志会被记录。


### unsubscribe

`unsubscribe` 允许用户取消通道的订阅。`params`是一个包含下列2个属性的对象: `channel` 和 `user` (是你想取消订阅的客户端的用户ID)

```javascript
{
    "method": "unsubscribe",
    "params": {
        "channel": "CHANNEL NAME",
        "user": "USER ID"
    }
}
```

响应样例:

```javascript
{
    "body": null,
    "error": null,
    "method": "unsubscribe"
}
```


### disconnect

`disconnect`允许使用用户ID来取消连接。`params`是一个带`user`属性的对象.

```javascript
{
    "method": "disconnect",
    "params": {
        "user": "USER ID"
    }
}
```

响应样例:

```javascript
{
    "body": null,
    "error": null,
    "method": "disconnect"
}
```


### presence

`presence`允许你获取通道当前的在线信息（所有订阅了这个通道的客户端）。`params`是一个带`channel`属性的对象.

```javascript
{
    "method": "presence",
    "params": {
        "channel": "CHANNEL NAME"
    }
}
```

响应样例:

```javascript
{
    "body": {
        "channel": "$public:chat",
        "data": {
            "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239": {
                "user": "2694",
                "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239",
                "default_info": {
                    "first_name": "Alexandr",
                    "last_name": "Emelin"
                },
                "channel_info": {
                    "channel_extra_info_example": "you can add additional JSON data when authorizing"
                }
            },
            "e5ee0ab0-fde1-4543-6f36-13f2201adeac": {
                "user": "2694",
                "client": "e5ee0ab0-fde1-4543-6f36-13f2201adeac",
                "default_info": {
                    "first_name": "Alexandr",
                    "last_name": "Emelin"
                },
                "channel_info": {
                    "channel_extra_info_example": "you can add additional JSON data when authorizing"
                }
            }
        }
    },
    "error": null,
    "method": "presence"
}
```


### history

`history`允许获取通道的历史信息 (最后发布到通道的消息列表).
`params`是一个带`channel`属性的对象.

```javascript
{
    "method": "history",
    "params": {
        "channel": "CHANNEL NAME"
    }
}
```

响应样例:

```javascript
{
    "body": {
        "channel": "$public:chat",
        "data": [
            {
                "uid": "8c5dca2e-1846-42e4-449e-682f615c4977",
                "timestamp": "1445536974",
                "info": {
                    "user": "2694",
                    "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239",
                    "default_info": {
                        "first_name": "Alexandr",
                        "last_name": "Emelin"
                    },
                    "channel_info": {
                        "channel_extra_info_example": "you can add additional JSON data when authorizing"
                    }
                },
                "channel": "$public:chat",
                "data": {
                    "input": "world"
                },
                "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239"
            },
            {
                "uid": "63ecba35-e9df-4dc6-4b72-a22f9c9f486f",
                "timestamp": "1445536969",
                "info": {
                    "user": "2694",
                    "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239",
                    "default_info": {
                        "first_name": "Alexandr",
                        "last_name": "Emelin"
                    },
                    "channel_info": {
                        "channel_extra_info_example": "you can add additional JSON data when authorizing"
                    }
                },
                "channel": "$public:chat",
                "data": {
                    "input": "hello"
                },
                "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239"
            }
        ]
    },
    "error": null,
    "method": "history"
}
```


### channels (Centrifugo >= 0.3.0)

`channels`允许获取当前活跃的通道列表（有1个或更多订阅者的）。

```javascript
{
    "method": "channels",
    "params": {}
}
```

响应样例:

```javascript
{
    "body": {
        "data": [
            "$public:chat",
            "news",
            "notifications"
        ]
    },
    "error": null,
    "method": "channels"
}
```

### stats (Centrifugo >= 1.0.0)

`stats` 获取Centrifugo节点的统计信息方法。

```javascript
{
    "method": "stats",
    "params": {}
}
```

响应样例:

```javascript
{
    "body": {
        "data": {
            "nodes": [
                {
                    "uid": "6045438c-1b65-4b86-79ee-0c35367f29a9",
                    "name": "MacAir.local_8000",
                    "num_goroutine": 21,
                    "num_clients": 0,
                    "num_unique_clients": 0,
                    "num_channels": 0,
                    "started_at": 1445536564,
                    "gomaxprocs": 1,
                    "num_cpu": 4,
                    "num_msg_published": 0,
                    "num_msg_queued": 0,
                    "num_msg_sent": 0,
                    "num_api_requests": 0,
                    "num_client_requests": 0,
                    "bytes_client_in": 0,
                    "bytes_client_out": 0,
                    "memory_sys": 7444728,
                    "cpu_usage": 0
                }
            ],
            "metrics_interval": 60
        }
    },
    "error": null,
    "method": "stats"
}
```

### node (Centrifugo >= 1.4.0)

`node`允许获取单个Centrifugo节点的信息。这些信息包含未聚合的计数(`stats`默认显示聚合的)。所以如果你要自行统计时间段的信息会特别有用。但这个方法你要单独请求你需要获取的节点。看[issue](https://github.com/centrifugal/centrifugo/issues/68)获取更多信息。

```javascript
{
    "method": "node",
    "params": {}
}
```

响应样例:

```
{
    "body": {
        "data":{
            "uid":"c3ceab87-8060-4c25-9cb4-94eb9db7899a",
            "name":"MacAir.local_8000",
            "num_goroutine":14,
            "num_clients":0,
            "num_unique_clients":0,
            "num_channels":0,
            "started_at":1455450238,
            "gomaxprocs":4,
            "num_cpu":4,
            "num_msg_published":0,
            "num_msg_queued":0,
            "num_msg_sent":0,
            "num_api_requests":3,
            "num_client_requests":0,
            "bytes_client_in":0,
            "bytes_client_out":0,
            "memory_sys":0,
            "cpu_usage":0
        }
    },
    "error":null,
    "method":"node"
}
```

注意，目前对Python, Ruby, PHP来说已经有现成的API，你无须自行构建。同时，这些现有的API也有助于你在其它语言中实现HTTP的API。
