# Admin的网页管理界面

Admin的网页管理界面[代码库](https://github.com/centrifugal/web)，1个基于ReactJS的单页应用，但内嵌在Centrifugo服务中，不需要再单独下载.

[Heroku上的演示站点](https://centrifugo.herokuapp.com) (密码 `demo`) 可以了解详细的使用场景.

![Admin的网页管理界面截图](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/web.gif)

功能有:

* 显示当前服务器的常规信息和节点统计.
* 实时监控通道的消息(通道的`watch`参数必须启用).
* 调用 `publish`, `unsubscribe`, `disconnect`, `history`, `presence`, `channels`, `stats`等命令.

要启用它，需要在调用`centrifugo`时带 `--web`参数.

```
centrifugo --config=config.json --admin --web
```

`--admin`启用管理websocket用于Admin的网页管理界面使用.

`--web`告诉Centrifugo启用Admin的网页管理界面.

**注意，你可以仅使用`--web`同时启用上述2项.**

当然，你需要额外设置2个参数: `admin_password` 和 `admin_secret`.

`config.json`

```json
{
    ...,
    "admin_password": "strong_password_to_log_in",
    "admin_secret": "strong_secret_key_to_sign_authorization_token"
}
```

* `admin_password` – 登录Admin的网页管理界面的密码
* `admin_secret` - 在Admin的网页管理界面调用api的加密串.

你也可以指定自己的Admin的网页管理界面:

```
centrifugo --config=config.json --admin --web --web_path=/path/to/web/app
```

当被防火墙拦截或是遇到问题时，也可以使用非安全方式（不建议）:

```
centrifugo --config=config.json --admin --insecure_admin --web
```
