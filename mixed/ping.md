# Ping消息

用于保持长连接，从Centrifugo v1.6.0 开始`centrifuge-js`发送定时间隔的ping消息到Centrifugo服务器，只有在连接空闲的时候发送。Javascript客户端库允许你禁用自动ping消息。

客户端到服务器端的ping可以检测连接的问题，而服务器端到客户端的ping（每隔25秒）可以关闭非活动的连接，比如检测到特别的PING websocket桢, 或是SockJS 的`h` 桢. 
简单说，如果你使用默认设置，你不用去管它。
