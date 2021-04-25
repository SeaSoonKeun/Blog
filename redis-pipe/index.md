# 

# redis-pipe

## 1. Why

Redis是一种基于**客户端-服务端模型**以及**请求/响应协议的TCP服务**。

这意味着通常情况下一个请求会遵循以下步骤：

- 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以**阻塞模式**，等待服务端响应。
- 服务端处理命令，并将结果返回给客户端。

客户端和服务器通过网络进行连接。这个连接可以很快（loopback接口）或**很慢**（建立了一个多次跳转的网络连接）

**RTT** (Round Trip Time - 往返时间). 当客户端需要在一个**批处理**中执行多次请求时很容易看到这是如何**影响性能**

## 2. What

将*多个命令*发送到服务器，而不用等待回复，最后在一个步骤中读取该答复。

一种几十年来广泛使用的技术。

例如许多POP3协议已经实现支持这个功能，大大加快了从服务器下载新邮件的过程。

## 3.How

### 管道（Pipelining） VS 脚本（Scripting）
```bash
[root@localhost ~]# echo -e "ping\nping\nping" | nc 127.0.0.1 6379
+PONG
+PONG
+PONG
```

```bash
ncat - Concatenate and redirect sockets
```
大量 pipeline 应用场景可通过 **Redis [脚本](http://redis.cn/commands/eval.html)**（Redis 版本 >= 2.6）得到更高效的处理，后者在服务器端执行大量工作。脚本的一大优势是可通过**最小的延迟读写数据，让读、计算、写等操作变得非常快**（pipeline 在这种情况下不能使用，因为客户端在写命令前需要读命令返回的结果）。

应用程序有时可能在 pipeline 中发送 [EVAL](http://redis.cn/commands/eval.html) 或 [EVALSHA](http://redis.cn/commands/evalsha.html) 命令。Redis 通过 [SCRIPT LOAD](http://redis.cn/commands/script-load.html) 命令（保证 EVALSHA 成功被调用）明确支持这种情况。








