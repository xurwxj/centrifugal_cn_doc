# 服务器状态统计

你可以通过`stats` API命令获取到服务器的状态统计信息，返回格式如下:

```javascript
"body": {
    "data": {
        "nodes": [
            {
                "uid": "8a7cf241-62c3-4ef5-8a20-83fbadb74c6f",
                "name": "Alexanders-MacBook-Pro.local_8000",
                "started_at": 1480764329,
                "metrics": {
                    "client_api_15_count": 0,
                    "client_api_15_microseconds_50%ile": 0,
                    "client_api_15_microseconds_90%ile": 0,
                    "client_api_15_microseconds_99%ile": 0,
                    "client_api_15_microseconds_99.99%ile": 0,
                    "client_api_15_microseconds_max": 0,
                    "client_api_15_microseconds_mean": 0,
                    "client_api_15_microseconds_min": 0,
                    "client_api_1_count": 0,
                    "client_api_1_microseconds_50%ile": 0,
                    "client_api_1_microseconds_90%ile": 0,
                    "client_api_1_microseconds_99%ile": 0,
                    "client_api_1_microseconds_99.99%ile": 0,
                    "client_api_1_microseconds_max": 0,
                    "client_api_1_microseconds_mean": 0,
                    "client_api_1_microseconds_min": 0,
                    "client_api_num_requests": 0,
                    "client_bytes_in": 0,
                    "client_bytes_out": 0,
                    "client_num_connect": 0,
                    "client_num_msg_published": 0,
                    "client_num_msg_queued": 0,
                    "client_num_msg_sent": 0,
                    "client_num_subscribe": 0,
                    "http_api_15_count": 0,
                    "http_api_15_microseconds_50%ile": 0,
                    "http_api_15_microseconds_90%ile": 0,
                    "http_api_15_microseconds_99%ile": 0,
                    "http_api_15_microseconds_99.99%ile": 0,
                    "http_api_15_microseconds_max": 0,
                    "http_api_15_microseconds_mean": 0,
                    "http_api_15_microseconds_min": 0,
                    "http_api_1_count": 0,
                    "http_api_1_microseconds_50%ile": 0,
                    "http_api_1_microseconds_90%ile": 0,
                    "http_api_1_microseconds_99%ile": 0,
                    "http_api_1_microseconds_99.99%ile": 0,
                    "http_api_1_microseconds_max": 0,
                    "http_api_1_microseconds_mean": 0,
                    "http_api_1_microseconds_min": 0,
                    "http_api_num_requests": 0,
                    "http_raw_ws_num_requests": 0,
                    "http_sockjs_num_requests": 0,
                    "node_cpu_usage": 0,
                    "node_memory_sys": 10524920,
                    "node_num_add_client_conn": 0,
                    "node_num_add_client_sub": 0,
                    "node_num_add_presence": 0,
                    "node_num_admin_msg_published": 0,
                    "node_num_admin_msg_received": 0,
                    "node_num_channels": 0,
                    "node_num_client_msg_published": 0,
                    "node_num_client_msg_received": 0,
                    "node_num_clients": 0,
                    "node_num_control_msg_published": 21,
                    "node_num_control_msg_received": 21,
                    "node_num_goroutine": 12,
                    "node_num_history": 0,
                    "node_num_join_msg_published": 0,
                    "node_num_join_msg_received": 0,
                    "node_num_last_message_id": 0,
                    "node_num_leave_msg_published": 0,
                    "node_num_leave_msg_received": 0,
                    "node_num_presence": 0,
                    "node_num_remove_client_conn": 0,
                    "node_num_remove_client_sub": 0,
                    "node_num_remove_presence": 0,
                    "node_num_unique_clients": 0,
                    "node_uptime_seconds": 120,
                }
            }
        ],
        "metrics_interval": 60
    }
}
```

默认Centrifugo每1分钟聚合这些信息，你可以通过`node_metrics_interval`配置参数进行调整。

"nodes"是每个正在运行的Centrifugo节点状态统计信息数组，每个数组元素对象包括:

`uid` – 节点的唯一id

`name` – 节点名

`started_at` – 节点启动的时间，以秒为单位的unix时间戳

`metrics` 是一系列值的map (keys总是为 `string`类型, values总是为 `integer`类型) - 注意，这个只是快照，只有在`node_metrics_interval`时间点后才变化.


一些值说明:

`node_memory_sys` – 节点内存使用数，单位是字节

`node_cpu_usage` – 节点cpu使用百分比

`node_num_goroutine` – goroutines的活跃数量 (Golang特有的)

`node_num_clients` – 客户端的连接数量

`node_num_unique_clients` – 唯一客户端的数量 (使用不同的用户ID)

`node_num_channels` – 活跃通道数量(至少1个订阅者)

`node_num_client_msg_published` – 消息发布的数量

`node_uptime_seconds` – 已经运行的时间，以秒为单位

`http_api_num_requests` – HTTP API请求数量

`client_api_num_requests` – 客户端API的请求数量

`client_bytes_in` – 客户端通过客户端API发送的字节数

`client_bytes_out` – 发送到客户端的字节数

`client_num_msg_queued` – 进入客户端队列的消息数(包括协议消息，加入/离开消息等)

`client_num_msg_sent` – 实际发到客户端的消息数(正常情况下是跟`num_msg_queued`值一样的)

也还有HDR的柱状图值: 用于HTTP API和client API。他们是1~15个周期，每个周期是每个`node_metric_interval`秒数，所以默认就是1~15分钟。
