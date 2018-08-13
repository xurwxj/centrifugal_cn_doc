# TLS证书

TLS/SSL 层对于安全或代理来说都很重要，这里有2种方式来实现: 
- 使用TLS `cert` 和 `key` 文件
- 使用自动生成的证书[ACME](https://ietf-wg-acme.github.io/acme/) ，当前只有[Let's Encrypt](https://letsencrypt.org/) 提供.

### 使用 crt 和 key 文件

如果要生成自定义证书，[请参照](https://devcenter.heroku.com/articles/ssl-certificate-self)。启动Centrifugo时使用以下参数:

```
./centrifugo --config=config.json --ssl --ssl_key=server.key --ssl_cert=server.crt
```

或者在配置文件中设置:

```json
{
  ...
  "ssl": true,
  "ssl_key": "server.key",
  "ssl_cert": "server.crt"
}
```

然后运行:

```
./centrifugo --config=config.json
```

### 自动证书

可在配置中设置Let's Encrypt相关信息:

```
{
  ...
  "ssl_autocert": true,
  "ssl_autocert_host_whitelist": "www.example.com",
  "ssl_autocert_cache_dir": "/tmp/certs",
  "ssl_autocert_email": "user@example.com"
}
```

`ssl_autocert`是否启用自动证书获取.

`ssl_autocert_host_whitelist` 证书对应的域名，这里可以以逗号分隔输入多个域名.

`ssl_autocert_cache_dir` 证书保存的缓存目录.

`ssl_autocert_email` 可选，用于发证机构与你联系.

当配置正确后，启动就能启用合适的证书，并且这个证书是会自动更新的。
另外要注意的是从v1.6.5开始，允许Centrifugo来支持TLS的客户端连接，比如一些旧的浏览器兼容，象Chrome 49 on Windows XP and IE8 on XP:

* `ssl_autocert_force_rsa` - 布尔值，默认是`false`。启用后会自动通过2048-bit RSA进行认证.
* `ssl_autocert_server_name` - 字符串，用于握手时的服务器名称指定，对于旧浏览器没有SNI支持的特别有用 - [这里查看](https://github.com/centrifugal/centrifugo/issues/144#issuecomment-279393819)
