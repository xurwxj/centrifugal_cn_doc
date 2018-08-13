# Websocket压缩

这里有个专业特性`permessage-deflate`可以用于websocket消息压缩，可以看下[好文章](https://www.igvita.com/2013/11/27/configuring-and-optimizing-websocket-compression/) .之所以是专业特性，是因为在[Gorilla Websocket](https://github.com/gorilla/websocket) 库中它本身就是一个需要专业知识积累才能使用的.

Websocket压缩可以减少流量，但在Centrifugo运行性能上会降低，要启用它，可以在配置文件中设置`websocket_compression: true`，但要注意启用后并不意味着每个连接或消息都会使用，那个取决于客户端的支持度. 另外1个参数是`websocket_compression_min_size`. 默认是0.这个值表示会根据客户端的支持程度最大可能去压缩每个消息.也可以根据[compress/flate级别](https://golang.org/pkg/compress/flate/#NewWriter)来指定压缩级别，如果要设置的话，可以使用`websocket_compression_level` (Centrifugo v1.6.4后的版本支持)参数来设置.
