# 高级配置

Centrifugo有些配置参数是适用于大部分应用或场景的，基本无须更改，这个章节会讲述这些参数：

#### client_channel_limit

默认值：128

设置单个客户端可以订阅的最大通道数量，在v1.6.0之前默认值是100.

#### max_channel_length

默认值：255

设置最长的通道名称长度.

#### user_connection_limit

默认值：0

单节点最多已知ID的用户数量，默认是无限的，0就是无限.

#### node_metrics_interval

默认值：60

Centrifugo聚合各种数据的间隔时间，默认60秒.

#### client_request_max_size

默认值：65536

客户端最大的请求内容大小，单位是字节.

#### client_queue_max_size

默认值：10485760

客户端最大的消息队列尺寸，单位是字节，超过这个值就关闭缓慢的读取连接，默认值是10兆.

#### sockjs_heartbeat_delay

默认值：0

发送SockJS h-frames的间隔时间，以秒为单位，从v1.6.0开始我们不再使用SockJS心跳帧，使用了客户端到服务器端的ping来替代。
frames as we use client to server pings.

#### websocket_compression

默认值：false

启用websocket压缩，具体请看相关章节.

#### gomaxprocs

默认值：0

默认Centrifugo使用所有可用的CPU核，但如果你要限制，可以变更这个值，注意不要超过你的cpu核数.

## 高级接入配置.

当你启动Centrifugo后，你有多个接入方式已经可用，默认至少有以下3种接入方式：

#### 默认接入.

SockJS接入 - 它需要客户端使用SockJS库:

```
http://localhost:8000/connection
```

通过纯Websocket协议的接入方式:

```
ws://localhost:8000/connection/websocket
```

API接入方式来`publish`消息到通道（也支持其它可用的API命令）:

```
http://localhost:8000/api/
```

默认所有接入方式都工作在端口`8000`上，你可以在配置中变更:

```
{
    "port": "9000"
}
```

在正式环境，请用你的域名来替换上面的 `localhost`。如果你的Centrifugo在代理或负载均衡后面，你可以在接入地址中不需要加入端口，只需要在域名后面加上上面的URL地址: `/connection`, `/connection/websocket`, `/api/`.

如果有需要，请自行调整上述接入方式.

#### 管理接入.

首先在配置中要启用管理接入:

```
{
    ...
    "admin": true,
    "admin_password": "password",
    "admin_secret": "secret"
}
```

启用后下面的接入地址才可用:

```
ws://localhost:8000/socket
```

这是一个管理方式的websocket连接，在大部分场景下，它只被我们内置的web界面使用，下面我们只说明如果启用这个界面:

```
{
    ...
    "web": true,
    "admin": true,
    "admin_password": "password",
    "admin_secret": "secret"
}
```

之后你就可以访问:

```
http://localhost:8000/
```

你需要使用`admin_password`的值登录.


#### 调试接入.

如果我们要对Centrifugo进行调试，可以参照下面的配置启动调试模式:

```
{
    ...
    "debug": true
}
```

得到的调试接入地址如:

```
http://localhost:8000/debug/pprof/
```

这个地址将会展示1个Centrifugo实例的内部状态，当遇到问题的时候可能特别有用。

#### 自定义管理和API端口

我们强烈建议不要暴露管理站点、调试和API接入方式。自定义在一定程度上可以加强保护，通过设置`admin_port`参数来保护管理站点:

```
{
    ...
    "admin_port": "10000"
}
```

地址会变成:
 
```
ws://localhost:10000/socket
```

调试地址变成:

```
http://localhost:10000/debug/pprof/
```

使用`api_port`参数来配置API接入方式的端口:

```
{
    ...
    "api_port": "10001"
}
```

API请求地址变成:

```
http://localhost:10001/api/
```
