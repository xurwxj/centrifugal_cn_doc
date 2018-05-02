# Tokens和签名

Centrifugo使用SHA-256 HMAC格式来创建连接Token，并用于服务器或客户端API与Centrifugo通信时的数据签名。

本章节将介绍如何为不同的行为创建Tokens和签名。如果你使用Python，则所有方法在`Cent`库中是现成的。如果你想创建自己的库，可以仔细了解本章节。

先讲连接Token.

### 客户端连接token

当客户端通过浏览器连接Centrifugo时，需要提供以下连接参数: `user`, `timestamp`, `info` (可选)和 `token`.

我们已经在其它章节说明了这些参数，这里我们将介绍如何生成一个token。

你需要做的是，使用Centrifugo的`secret`通过HMAC的hex数字加密方式来生成，并且混合`user`（是用户ID）, `timestamp`和可选的`info`
(注意顺序也是要这样的，顺序是很重要的). 可选的`info`意味着在生成的时候是可以忽略的，它不会影响加密结果。

我们来看一下代码样例:

```python
import six
import hmac
from hashlib import sha256

def generate_token(secret, user, timestamp, info=""):
    sign = hmac.new(six.b(secret), digestmod=sha256)
    sign.update(six.b(user))
    sign.update(six.b(timestamp))
    sign.update(six.b(info))
    return sign.hexdigest()
```


### 私有通道订阅签名

当客户端订阅一个私有通道时，js客户端发送的AJAX POST请求必须要包含`client` ID及1个或多个的私有`channels`。返回通道作为属性的对象作为响应。

比如，如果你接到通道`$one` 和 `$two`的请求，你需要返回以下内容:

```python
{
    "$one": {
        "info": "{}",
        "sign": "ssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss"
    },
    "$two": {
        "info": "{}",
        "sign": "ssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss"
    }
}
```

注意： `info`是这个通道连接的额外信息，而`sign`是基于客户端ID、通道名和通道信息构建的HMAC。 下面是Python代码示例:

```python
import six
import hmac
from hashlib import sha256

def generate_channel_sign(secret, client, channel, info=""):
    auth = hmac.new(six.b(secret), digestmod=sha256)
    auth.update(six.b(str(client)))
    auth.update(six.b(str(channel)))
    auth.update(six.b(info))
    return auth.hexdigest()
```

从代码中可以看到info是已经被编码的JSON字符串。


### HTTP API请求签名

当你使用Centrifugo服务器API的时候，每次请求你都需要提供签名，下面是Python的代码示例:

```python
import six
import hmac
from hashlib import sha256

def generate_api_sign(self, secret, encoded_data):
    sign = hmac.new(six.b(secret), digestmod=sha256)
    sign.update(six.b(encoded_data))
    return sign.hexdigest()
```

`encoded_data`是带API命令的JSON字符串。
