# 连接检查

如果客户端通过合适的方式连接到了Centrifugo，则每个连接可以保持长连接。如果客户端已经连接到Centrifugo，哪怕你在你的应用中禁用了这个用户，他还是能从已经订阅的通道中读取信息。在某些情况下，我们不希望这样。

配置文件中有一个特殊的选项: `connection_lifetime`. 默认值是`0`表示不启用连接检查机制。如果设置比`0`大的值，则表示每隔多少秒就对连接进行检查验证。如果连接时间过期，则javascript浏览器客户端会发起1个AJAX POST请求，默认是请求到`/centrifuge/refresh/`，你可以通过`refreshEndpoint`参数设置到其它地址。如果是自定义的地址，请务必要返回带连接验证的JSON。比如在python中你可以这样写:

```python
    to_return = {
        'user': "USER ID,
        'timestamp': "CURRENT TIMESTAMP AS INTEGER",
        'info': "ADDITIONAL CONNECTION INFO",
        'token': "TOKEN BASED ON PARAMS ABOVE",
    }
    return json.dumps(to_return)
```

你返回的连接验证必须要跟`user`渲染页面时的值一样，但使用当前的`timestamp`时间戳。Javascript客户端会把它们发给Centrifugo服务器，然后连接将自动刷新为新的周期。

如果你不想刷新，则可以直接返回403即可。