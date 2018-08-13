# 浏览器客户端的私有通道

所有`$`开头的通道都被认为是私有通道，默认javascript客户端会发送AJAX POST请求到`/centrifuge/auth/`，你可以通过改变设置(`authEndpoint` 和 `authHeaders`)来调整url.

POST请求是1个JSON 对象，主要包括2个内容: `client`和 `channels`. Client是当前客户端ID的字符串，channels是当前客户端订阅的通道信息:

```javascript
{
    "client": "CLIENT ID",
    "channels": ["$one"]
}
```

假设客户端订阅2个私有通道: ``$one`` 和 ``$two``:

```javascript
centrifuge.subscribe('$one', function(message) {
    // process message
});

centrifuge.subscribe('$two', function(message) {
    // process message
});
```

一般来说Centrifuge会发送2个分开的POST请求，当然 也可以使用`startAuthBatching` 和 `stopAuthBatching`来一次性发送:

```javascript
centrifuge.startAuthBatching();

centrifuge.subscribe('$one', function(message) {
    // process message
});

centrifuge.subscribe('$two', function(message) {
    // process message
});

centrifuge.stopAuthBatching();
```

当一次性发送时，发送的内容结构就如：

```javascript
{
    "client": "CLIENT ID",
    "channels": ["$one", "$two"]
}
```

而返回值则如下:

```
{
    "$one": {
        "sign": "PRIVATE SIGN",
        "info": ""
    }
}
```

* `sign` – 私有通道的订阅签名
* `info` – 用于客户端发送消息的`channel_info` ，一般是没有用途的.

查看[Tokens和签名](../server/tokens_and_signatures.md) 来查看如何创建签名

如果不允许订阅则返回:

```
{
    "$one": {
        "status": 403
    }
}
```

下面是基于Tornado实现的验证方式:

```python
from cent.core import generate_channel_sign

class CentrifugeAuthHandler(tornado.web.RequestHandler):

    def check_xsrf_cookie(self):
        pass

    def post(self):

        try:
            data = json.loads(self.request.body)
        except ValueError:
            raise tornado.web.HTTPError(403)

        client = data.get("client", "")
        channels = data.get("channels", [])

        to_return = {}

        for channel in channels:
            info = json.dumps({
                'channel_extra_info_example': 'you can add additional JSON data when authorizing'
            })
            sign = generate_channel_sign(
                options.secret_key, client, channel, info=info
            )
            to_return[channel] = {
                "sign": sign,
                "info": info
            }

        self.set_header('Content-Type', 'application/json; charset="utf-8"')
        self.write(json.dumps(to_return))
```

下面是403禁用的代码样例:

```python
from cent.core import generate_channel_sign

class CentrifugeAuthHandler(tornado.web.RequestHandler):

    def check_xsrf_cookie(self):
        pass

    def post(self):

        try:
            data = json.loads(self.request.body)
        except ValueError:
            raise tornado.web.HTTPError(403)

        client = data.get("client", "")
        channels = data.get("channels", [])

        to_return = {}

        for channel in channels:
            sign = generate_channel_sign(
                options.secret_key, client, channel
            )
            to_return[channel] = {
                "status": 403,
            }

        self.set_header('Content-Type', 'application/json; charset="utf-8"')
        self.write(json.dumps(to_return))
```

如果用户不活跃时，可以直接返回403:

```python
from cent.core import generate_channel_sign

class CentrifugeAuthHandler(tornado.web.RequestHandler):

    def check_xsrf_cookie(self):
        pass

    def post(self):
        raise tornado.web.HTTPError(403)
```

这将阻止用户订阅任何私有通道.

如果你使用PHP (特别是使用Laravel框架)时，可以参照[该代码样例](https://gist.github.com/Malezha/a9bdfbddee15bfd624d4) .
