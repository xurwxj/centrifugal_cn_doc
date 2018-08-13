# Redis高可用

**1.4.2版本之后启用**

Centrifugo支持通过特殊的渠道来支持Redis高可用 [Sentinel](http://redis.io/topics/sentinel).

要启用，你只需设置2个参数: `redis_master_name` 和 `redis_sentinels`.

`redis_master_name` - 是Redis集群的名字.

`redis_sentinels` - 逗号分隔的Redis服务器地址，至少要1个.

启动方式如下:

```
centrifugo --config=config.json --engine=redis --redis_master_name=mymaster --redis_sentinels=":26379"
```

Sentinel配置文件看起来象:

```
port 26379
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 10000
sentinel failover-timeout mymaster 60000
```

你可以查找Sentinels [官方文档](http://redis.io/topics/sentinel).

注意，启用后，Redis的挂机时间取决于你Redis服务器设置的发现机制值.


