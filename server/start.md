# Centrifugo安装和快速指南

Go是一种完美的语言 - 它让开发者有机会在不同的操作系统上通过单一的可执行文件来实现应用。这意味着你只要直接根据你的系统下载Centrifugo - [下载最新发布版](https://github.com/centrifugal/centrifugo/releases) ，然后解压就完成了安装。

现在你可以看到Centrifugo的帮助信息:

```
./centrifugo -h
```

Centrifugo服务器节点要求一个带密钥的配置文件。
如果你是刚刚开始使用Centrifugo，你可以执行`genconfig`命令来生成一个最小化的配置文件：

```bash
./centrifugo genconfig
```

上面的命令会在当前目录自动生成一个带密钥的配置文件`config.json`，你可以直接用它来跑Centrifugo服务器:

```bash
./centrifugo --config=config.json
```

我们将在下一节详细讨论服务器的各项配置。

你可以把`centrifugo`放到你系统的`bin`目录，或者是链接到可执行目录，这样就可以随时运行：

```bash
centrifugo --config=config.json
```

在生产环境，你最好把Centrifugo设置成后台进程或服务。我们已经为大部分受欢迎的Linux系统预编译了rpm和deb格式的安装包，并提供了一个Docker镜像，在后面的部署章节，你可以看到详细的说明。
