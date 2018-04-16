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

* `anonymous` – this option enables anonymous access (with empty user ID in connection parameters). In most situations your application works with authorized users so every user has its own unique id. But if you provide real-time features for public access you may need unauthorized access to some channels. Turn on this option and use empty string as user ID. By default `false`.

* `presence` – enable/disable presence information. Presence is a structure with clients currently subscribed on channel. By default `false` – i.e. no presence information available for channels.

* `join_leave` – enable/disable sending join(leave) messages when client subscribes on channel (unsubscribes from channel). By default `false`.

* `history_size` – history size (amount of messages) for channels. As Centrifugo keeps all history messages in memory it's very important to limit maximum amount of messages in channel history to reasonable minimum. By default history size is `0` - this means that channels will have no history messages at all. As soon as history enabled then `history_size` defines maximum amount of messages that Centrifugo will keep for **each** channel in namespace during history lifetime (see below).

* `history_lifetime` – interval in seconds how long to keep channel history messages. As all history is storing in memory it is also very important to get rid of old history data for unused (inactive for a long time) channels. By default history lifetime is `0` – this means that channels will have no history messages at all. **So to get history messages you should wisely configure both `history_size` and `history_lifetime` options**.

* `recover` (**new in v1.2.0**) – boolean option, when enabled Centrifugo will try to recover missed messages published while client was disconnected for some reason (bad internet connection for example). By default `false`. This option must be used in conjunction with reasonably configured message history for channel i.e. `history_size` and `history_lifetime` **must be set** (because Centrifugo uses channel message history to recover messages). Also note that note all real-time events require this feature turned on so think wisely when you need this. See more details about how this option works in [special chapter](recover.md).

* `history_drop_inactive` (**new in v1.3.0**) – boolean option, allows to drastically reduce resource usage (engine memory usage, messages travelling around) when you use message history for channels. In couple of words when enabled Centrifugo will drop history messages that no one needs. Please, see [issue on Github](https://github.com/centrifugal/centrifugo/issues/50) to get more information about option use case scenario and edge cases it involves.

Let's look how to set all of these options in config:

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

And the last channel specific option is `namespaces`. `namespaces` are optional and if set must be an array of namespace objects. Namespace allows to configure custom options for channels starting with namespace name. This provides a great control over channel behaviour.

Namespace has a name and the same channel options (with same defaults) as described above.

* `name` - unique namespace name (name must must consist of letters, numbers, underscores or hyphens and be more than 2 symbols length i.e. satisfy regexp `^[-a-zA-Z0-9_]{2,}$`).

If you want to use namespace options for channel - you must include namespace name into
channel name with `:` as separator:

`public:messages`

`gossips:messages`

Where `public` and `gossips` are namespace names from project `namespaces`.

All things together here is an example of `config.json` which includes registered project with all options set and 2 additional namespaces in it:

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

Channel `news` will use global project options.

Channel `public:news` will use `public` namespace's options.

Channel `gossips:news` will use `gossips` namespace's options.
