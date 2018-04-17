# 重要配置说明

正如我在前面章节所说，配置文件必须至少包括一个`secret`选项。

最小化的配置文件看起来如下面一样:

```javascript
{
    "secret": "very-long-secret-key"
}
```

**要确保只有Centrifugo和你后端的应用知道这个密钥**。它用于生成客户端连接tokens (在后面会有更详细的说明), 认证API请求和私有通道订阅请求。

下一个是`connection_lifetime`:

`connection_lifetime` – 这是客户端的过期时间设置，以秒为单位。默认是`0` - 这表示长连接，永远不会过期。在后面的章节请仔细查看连接过期机制。在大部分情况下，你不需要变更这个设置。

```javascript
{
    "secret": "very-long-secret-key",
    "connection_lifetime": 0
}
```

通道是客户端用于订阅和发布消息的实体。通道仅是一个字符串，但不同的符号代表不同的意思 - 在后面可以看到更多关于通道的说明。下面介绍会影响通道的相关配置项:

* `watch` – 这个选项会使Centrifugo把消息发布到管理通道（这些消息可以在管理网站的`messages tab`看到). 默认值是 `false`. 注意，启用这个选项要小心，因为高频率的消息有可能导致管理客户端无法处理所有消息，建议只有在开发环境启用或者是选择一些通道使用合理的消息频率.

* `publish` – 允许客户端直接发布消息。你的应用永远不会接收这些消息。正常情况下，所有消息应该是由你的应用后端使用Centrifugo API发布，但这个选项对于想构建没有后端验证、保存数据库需要的时候特别有用，对于演示和练习实时想法也非常有用。但要注意，客户端只有先订阅通道成功，才能发布消息至通道。默认值是`false`.

* `anonymous` – 允许匿名接入（意味着user ID是空的）。大部分情况下，你的用户通过其唯一id验证接入，但你也可以开放某些通道允许公开接入，以启用实时特性。默认值是`false`.

* `presence` – 启用/禁用在线状态信息。启用时可获取到通道当前订阅的客户端。默认值是`false` – 表示不启用.

* `join_leave` – 启用/禁用客户端订阅或取消订阅通道的触发消息，默认值是`false`。启用时可用于一些业务端的特殊逻辑处理。

* `history_size` – 通道的历史消息最大保留数。Centrifugo默认保留所有消息在内存中，所以通道历史消息最大保留数设置为一个合理的最小值很重要，它与下面的`history_lifetime`值一起使用。默认值是`0` - 表示通道不保留任何历史消息。一旦设置为大于0的值，则表示在下面的`history_lifetime`值期间，在命名空间中的通道都将保留这个数量的历史消息。

* `history_lifetime` – 通道历史消息保留时间，超过这个时间的历史消息都将从内存中清除。默认值是`0` – 表示通道不保留任何历史消息。**如果你想要历史消息，你必须同时设置`history_size` 和 `history_lifetime` **.

* `recover` (**v1.2.0开始启用**) – 布尔值选项，启用时Centrifugo会在客户端断开连接时尝试恢复消息。默认值是`false`。这个选项必须要搭配`history_size`和 `history_lifetime`一起使用(因为Centrifugo是从历史消息中进行恢复操作). 同时所有实时事件都需要这个选项启用，更多信息请看[特殊章节](recover.md).

* `history_drop_inactive` (**v1.3.0开始启用**) – 布尔值选项，当启用历史消息设置时允许动态的调整资源使用（比如内存、消息保留等），换句话说，启用这个选项，可以让Centrifugo清理掉不再需要的历史消息，可以查看[issue on Github](https://github.com/centrifugal/centrifugo/issues/50) 了解更多使用场景和逻辑。

这些值在配置文件中的样例如下:

```javascript
{
    "secret": "very-long-secret-key",
    "connection_lifetime": 0,
    "anonymous": true,
    "publish": true,
    "watch": true,
    "presence": true,
    "join_leave": true,
    "history_size": 10,
    "history_lifetime": 30,
    "recover": true
}
```

最后要特别说明的与通道相关的选项是`namespaces`. `namespaces`是可选的，并且是1个对象数组，允许不同的命名空间对一批通道定制选项，可以实现不同的行为结果。

Namespace有1个名称及和其它通道选项一样的选项（如上面描述的）。

* `name` - 唯一的空间名称 (只能由字母、数字、下划线、中杠组成，不少于2个字符长度，满足`^[-a-zA-Z0-9_]{2,}$`的正则规则).

如果你想为通道启用命名空间，你必须要用`:`进行分隔:

`public:messages`

`gossips:messages`

`public` 和 `gossips`是 `namespaces`中的命名空间名.

所有选项组合在一起的配置文件样例如下:

```javascript
{
    "secret": "very-long-secret-key",
    "connection_lifetime": 0,
    "anonymous": true,
    "publish": true,
    "watch": true,
    "presence": true,
    "join_leave": true,
    "history_size": 10,
    "history_lifetime": 30,
    "namespaces": [
        {
          "name": "public",
          "publish": true,
          "presence": true,
          "join_leave": true,
          "anonymous": true,
          "history_size": 10,
          "history_lifetime": 30,
          "recover": true
        },
        {
          "name": "gossips",
          "watch": true
        }
    ]
}
```

通道 `news`将使用全局设置

通道 `public:news` 将使用命名空间`public`的设置

通道 `gossips:news` 将使用命名空间`gossips` 的设置
