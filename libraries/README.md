# Server端API库

Centrifugal已经拥有多种服务器端的api库，如果你没找到，你也可以通过Redis引擎或自行实现[参照文档](../server/api.md).

另外[Cent](https://github.com/centrifugal/cent)包含所有你需要的服务器端api，特别是对python来说.

### Python

Python HTTP API [api库](https://github.com/centrifugal/cent).

它支持命令行使用Centrifugo.

### PHP

我们有2个PHP的库.

一是Dmitriy Soldatenko创建的[HTTP API](https://github.com/sl4mmer/phpcent)， 使用非常简单.

另外一个是[Oleh Ozimok](https://github.com/oleh-ozimok)开发的[php-centrifugo](https://github.com/oleh-ozimok/php-centrifugo)， 使用Redis引擎.

### Ruby

Ruby[库](https://github.com/centrifugal/rubycent).


### NodeJS

NodeJS[库](https://github.com/centrifugal/jscent).

基本使用样例:

```javascript
Client = require("jscent");

var c = new Client({url: "http://localhost:8000", secret: "secret"});

c.publish("$public:chat", {"input": "test"}, function(err, resp){console.log(err, resp)});
```

### Go

Go[HTTP API库](https://github.com/centrifugal/gocent).


### Java

是第三方的，似乎并未保持持续的更新。

* [API library for Java](https://github.com/mcoetzee/centrifuge-publisher)，由Markus Coetzee创建

要跟Centrifugo客户端协同，注意要使用`sha256` HMAC算法而不是`md5`，并且不要使用project ID (project ID已经被废弃了).
