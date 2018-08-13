# 非安全模式

你可以让Centrifugo运行在非安全模式，但这个模式下会.

* 禁用客户端时间戳和token检查
* 允许匿名接入所有通道
* 允许客户端随意发布到任何通道
* 忽略连接检测

一般只适用于个人或演示场景，生产环境除非你清楚你在做什么.

### 服务器端

启动Centrifugo时使用`--insecure`参数:

```bash
centrifuge --config=config.json --insecure
```

也可以在配置文件中启用`insecure`.

### 客户端

连接样例:

```javascript
var centrifuge = new Centrifuge({
    "url": url,
    "insecure": true
});
```

[演示代码](https://github.com/centrifugal/centrifuge/tree/master/examples/insecure_mode) 

# 非安全的HTTP API方式

启动Centrifugo时带`--insecure_api`参数:

```bash
centrifugo --config=config.json --insecure_api
```

# 非安全的管理模式(v1.3.0起应用, v1.6.0时进行了调整)

允许不设置`admin_password`和 `admin_secret`就进入管理界面，注意 **应该仅限于开发或个人小环境时使用**.

启动Centrifugo时带`--insecure_admin`参数:

```bash
centrifugo --config=config.json --insecure_admin
```

```bash
centrifugo --config=config.json --insecure_admin --web
```

再次提醒: 不管任何哪种非安全模式，请不要在生产环境使用.
