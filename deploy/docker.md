# Docker镜像

Centrifugo拥有 [Docker镜像](https://hub.docker.com/r/centrifugo/centrifugo/).

```
docker pull centrifugo/centrifugo
```

运行:

```bash
docker run --ulimit nofile=65536:65536 -v /host/dir/with/config/file:/centrifugo -p 8000:8000 centrifugo/centrifugo centrifugo -c config.json
```

注意，允许在启动时设置nofile限制.

运行管理界面:

```bash
docker run --ulimit nofile=65536:65536 -v /host/dir/with/config/file:/centrifugo -p 8000:8000 centrifugo/centrifugo centrifugo -c config.json --web
```

