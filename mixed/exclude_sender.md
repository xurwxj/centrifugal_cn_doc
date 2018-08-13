# 根据发送人除外处理

有时你要阻止客户端在从通道中接收到消息后进行处理时使用，0.2.0之后才有的特性，样例代码如下:

```javascript
var subscription = centrifuge.subscribe(channel, function(message) {
    if (message.client === centrifuge.getClientId()) {
        return;
    }
    // if clients not equal – process message as usual
});
```
