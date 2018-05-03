# 信号处理

这个章节是讲从操作系统发送信息给Centrifugo。

### 通道参数和配置重载入

Centrifugo可以重载通道参数或其它的配置选项，通过发送`HUP`信号到Centrifugo进程:

```bash
kill -HUP <CENTRIFUGO_PROCESS_PID>
```

注意：部分参数或配置不能被重载，因为Centrifugo在启动过程中要使用，比如跟引擎相关的参数。