# 配置概述

Centrifugo支持JSON, TOML或者YAML格式的文件作为配置文件.
感谢viper库的支持 - [viper](https://github.com/spf13/viper).

首先请先了解所有可用的命令选项:

```bash
centrifugo -h
```

你会看到如下面输出的内容:

```
Centrifugo. Real-time messaging (Websockets or SockJS) server in Go.

Usage: 
   [flags]
   [command]
Available Commands: 
  version     Centrifugo版本号
  checkconfig 检查配置文件
  genconfig   生成简单的配置文件用于快速启动
  help        可用于获取任何命令的帮助信息

Flags:
  -a, --address string             监听的地址
      --admin                      启用管理网站
      --admin_port string          管理网站的端口（可选）
      --api_port string            api的端口（可选）
  -c, --config string              配置文件路径 (默认是 "config.json")
  -d, --debug                      启用调试模式
  -e, --engine string              引擎: memory 或 redis (默认 "memory")
      --insecure                   非安全模式启动
      --insecure_admin             使用非安全管理网站 – 意味着无须授权验证
      --insecure_api               使用非安全模式的API
      --log_file string            日志文件路径 - 如果不指定则输出到控制台（可选）
      --log_level string           日志级别: trace, debug, info, error, critical, fatal 或者 none (默认 "info")
  -n, --name string                唯一的节点名称
      --pid_file string            PID文件路径（可选）
  -p, --port string                HTTP 服务器端口(默认 "8000")
      --redis_api                  启用Redis API监听器 (Redis engine)
      --redis_api_num_shards int   redis API队列共享池大小 (Redis engine)
      --redis_db string            redis数据库 (Redis engine) (默认 "0")
      --redis_host string          redis服务器地址 (Redis engine) (默认 "127.0.0.1")
      --redis_master_name string   Redis主Sentinel监控数量 (Redis engine)
      --redis_password string      redis用户密码 (Redis engine)
      --redis_pool int             Redis连接池大小 (Redis engine) (默认 256)
      --redis_port string          redis端口 (Redis engine) (默认 "6379")
      --redis_sentinels string     以逗号分隔的Sentinels (Redis engine)
      --redis_url string           redis连接URL，格式是：redis://:password@hostname:port/db (Redis engine)
      --ssl                        启用SSL，这个要求X509认证和密钥文件
      --ssl_cert string            X509认证文件路径
      --ssl_key string             X509认证密钥路径
  -w, --web                        启用管理网站应用（注意：会自动启用管理端口）
      --web_path string            自定义管理网站应用路径（可选）

Global Flags:
  -h, --help   help for 

Use " help [command]" for more information about a command.
```

### 版本查看

执行下列命令查看当前版本:

```
centrifugo version
```

### JSON格式配置文件样例

注意JSON格式配置文件必须是有效的，符合JSON规范的.

下面是我个人用于开发Centrifugo的配置文件:

```javascript
{
  "secret": "secret",
  "namespaces": [
    {
      "name": "public",
      "publish": true,
      "watch": true,
      "presence": true,
      "join_leave": true,
      "history_size": 10,
      "history_lifetime": 30
    }
  ],
  "log_level": "debug"
}
```

只有 **secret** 这一项是必须的.

你可以在下面了解到`namespaces`的用处。

下面是**最小化要求的配置文件**:

```javascript
{
  "secret": "secret"
}
```

在生产环境请一定要使用足够强壮的密码

### TOML

Centrifugo也支持TOML格式的配置文件:

```
centrifugo --config=config.toml
```

`config.toml`配置样例:

```
log_level = "debug"

secret = "secret"

[[namespaces]]
    name = "public"
    publish = true
    watch = true
    presence = true
    join_leave = true
    history_size = 10
    history_lifetime = 30
```

各项配置与上面的JSON格式是一致的.

### YAML

也支持YAML格式的配置文件， `config.yaml`:

```
log_level: debug

secret: secret
namespaces:
  - name: public
    publish: true
    watch: true
    presence: true
    join_leave: true
    history_size: 10
    history_lifetime: 30
```

使用YAML格式时要注意使用空格而不是tab来写

### 多个项目

从Centrifugo 1.0.0 开始不支持多个项目。

### checkconfig

Centrifugo有一个特殊命令用于检测配置文件`checkconfig`:

```bash
centrifugo checkconfig --config=config.json
```

如果在验证期间发现错误 - 程序将退出把显示错误信息，退出状态值是1

### genconfig

用于生成简单配置文件的`genconfig`:

```
centrifugo genconfig -c config.json
```

它将自动在命令执行的当前目录生成一个简单的配置文件。

### 重要的命令行选项说明

在下一节，我们将详细讨论项目设置，但在开始之前先了解这些重要的命令行选项，这些选项用于配置Centrifugo正常运行：

* `--address` – 绑定Centrifugo到一个指定的地址(默认是空的 `""`)
* `--port` – 绑定Centrifugo到指定端口 (默认是`8000`)
* `--engine` – 使用的engine - `memory` 或者 `redis` (默认是 `memory`). 在后面了解更多关于engine的信息
* `--web` – 管理站点执行的目录
* `--name` – 指定当前Centrifugo服务器节点的名称 – 这是可选的，Centrifugo默认会使用主机名和端口作为节点名称

还有更多的选项我们会在后面讨论。注意所有命令行选项可以通过配置文件设置，但命令行选项值优先于配置文件中的值，这个可以查看[viper](https://github.com/spf13/viper) 了解更多配置项优先级的信息。
