# 系统优化

跟Centrifugo/Centrifuge相关的系统参数有以下.

### open files limit

获取当前的文件打开数量限制:

```
ulimit -n
```

查看http://docs.basho.com/riak/latest/ops/tuning/open-files-limit/了解如何调整.

如果默认安装Centrifugo，这个值会被设置成32768.

如果使用了Nginx，也请相应这个值.

### lots of sockets in TIME_WAIT state.

查看处于TIME_WAIT状态的连接.

```
netstat -an |grep TIME_WAIT | grep CENTRIFUGO_PID | wc -l
```

这里可以了解TIME_WAIT信息: http://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux.html

这里可以了解连接数很多时如何优化: https://engineering.gosquared.com/optimising-nginx-node-js-and-networking-for-heavy-workloads.

总结来说就是:

1. 增加 ip_local_port_range
2. 如果使用nginx，则在上行流中设置`keepalive`.

```
upstream centrifugo {
    #sticky;
    ip_hash;
    server 127.0.0.1:8000;
    keepalive 512;
}
```

3. 如果始终无法解决，则关注 `net.ipv4.tcp_tw_reuse`这一项
