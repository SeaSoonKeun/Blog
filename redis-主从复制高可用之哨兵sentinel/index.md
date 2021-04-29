# Redis-主从复制高可用之哨兵（Sentinel）


# Redis-主从复制高可用之哨兵（Sentinel）

## 1. 基本知识

### 1）哨兵的功能

- **监控（Monitoring**）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
- **提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

### 2）启动哨兵的两种方式

​	对于 redis-sentinel 程序， 你可以用以下命令来启动 Sentinel 系统：

​	对于 redis-server 程序， 你可以用以下命令来启动一个运行在 Sentinel 模式下的 Redis 服务器：

```
redis-server /path/to/sentinel.conf --sentinel
```

运行配置文件，sentinel自带配置文件

### 3）哨兵的配置文件解释

路径：`/opt/soft/redis/redis-6.0.6/sentinel.conf` 	# 前面为安装路径

默认配置如下

```bash
port 26379
daemonize no
pidfile /var/run/redis-sentinel.pid
logfile ""
dir /tmp
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
```

第一行配置指示 Sentinel 去监视一个名为 mymaster 的主服务器， 这个主服务器的 IP 地址为 127.0.0.1 ， 端口号为 6379 ， 而将这个主服务器判断为失效至少需要 2 个 Sentinel 同意 （只要同意 Sentinel 的数量不达标，自动故障迁移就不会执行）。

- down-after-milliseconds 选项指定了 Sentinel 认为服务器已经断线所需的毫秒数。

如果服务器在给定的毫秒数之内， 没有返回 Sentinel 发送的 PING 命令的回复， 或者返回一个错误， 那么 Sentinel 将这个服务器标记为**主观下线**（subjectively down，简称 SDOWN ）。

不过只有一个 Sentinel 将服务器标记为主观下线并不一定会引起服务器的自动故障迁移： 只有在足够数量的 Sentinel 都将一个服务器标记为主观下线之后， 服务器才会被标记为**客观下线**（objectively down， 简称 ODOWN ）， 这时自动故障迁移才会执行。

- parallel-syncs 选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步， 这个数字越小， 完成故障转移所需的时间就越长。

如果从服务器被设置为允许使用过期数据集（参见对 redis.conf 文件中对 slave-serve-stale-data 选项的说明）， 那么你可能不希望所有从服务器都在同一时间向新的主服务器发送同步请求， 因为尽管复制过程的绝大部分步骤都不会阻塞从服务器， 但从服务器在载入主服务器发来的 RDB 文件时， 仍然会造成从服务器在一段时间内不能处理命令请求： 如果全部从服务器一起对新的主服务器进行同步， 那么就可能会造成所有从服务器在短时间内全部不可用的情况出现。

你可以通过将这个值设为 1 来保证每次只有一个从服务器处于不能处理命令请求的状态。

### 4）主观下线和客观下线状态

- 主观下线（Subjectively Down， 简称 SDOWN）指的是单个 Sentinel 实例对服务器做出的下线判断。
- 客观下线（Objectively Down， 简称 ODOWN）指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断。 （一个 Sentinel 可以通过向另一个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令来询问对方是否认为给定的服务器已下线。）

如果一个服务器没有在 master-down-after-milliseconds 选项所指定的时间内， 对向它发送 PING 命令的 Sentinel 返回一个有效回复（valid reply）， 那么 Sentinel 就会将这个服务器标记为主观下线。

服务器对 PING 命令的有效回复可以是以下三种回复的其中一种：

- 返回 +PONG 。
- 返回 -LOADING 错误。
- 返回 -MASTERDOWN 错误。

如果服务器返回除以上三种回复之外的其他回复， 又或者在指定时间内没有回复 PING 命令， 那么 Sentinel 认为服务器返回的回复无效（non-valid）。



### 5）故障转移过程

一次故障转移操作由以下步骤组成：

- 发现主服务器已经进入客观下线状态。
- 对我们的当前纪元进行自增（详情请参考 Raft leader election ）， 并尝试在这个纪元中当选。
- 如果当选失败， 那么在设定的故障迁移超时时间的两倍之后， 重新尝试当选。 如果当选成功， 那么执行以下步骤。
- 选出一个从服务器，并将它升级为主服务器。
- 向被选中的从服务器发送 `SLAVEOF NO ONE` 命令，让它转变为主服务器。
- 通过发布与订阅功能， 将更新后的配置传播给所有其他 Sentinel ， 其他 Sentinel 对它们自己的配置进行更新。
- 向已下线主服务器的从服务器发送 [SLAVEOF](http://redis.cn/commands/slaveof.html) 命令， 让它们去复制新的主服务器。
- 当所有从服务器都已经开始复制新的主服务器时， 领头 Sentinel 终止这次故障迁移操作。

Sentinel 使用以下规则来选择**新的主服务器**：

- 在失效主服务器属下的从服务器当中， 那些被标记为主观下线、已断线、或者最后一次回复 [PING](http://redis.cn/commands/ping.html) 命令的时间大于五秒钟的从服务器都会被淘汰。
- 在失效主服务器属下的从服务器当中， 那些与失效主服务器连接断开的时长超过 down-after 选项指定的时长十倍的从服务器都会被淘汰。
- 在经历了以上两轮淘汰之后剩下来的从服务器中， 我们选出复制偏移量（replication offset）最大的那个从服务器作为新的主服务器； 如果复制偏移量不可用， 或者从服务器的复制偏移量相同， 那么带有最小运行 ID 的那个从服务器成为新的主服务器。

### 6）通过psubscribe观察sentinal订阅发布信息

客户端可以将 Sentinel 看作是一个只提供了订阅功能的 Redis 服务器： 你不可以使用 [PUBLISH](http://redis.cn/commands/publish.html) 命令向这个服务器发送信息， 但你可以用 [SUBSCRIBE](http://redis.cn/commands/subscribe.html) 命令或者 [PSUBSCRIBE](http://redis.cn/commands/psubscribe.html) 命令， 通过订阅给定的频道来获取相应的事件提醒。

一个频道能够接收和这个频道的名字相同的事件。 比如说， 名为 +sdown 的频道就可以接收所有实例进入主观下线（SDOWN）状态的事件。

通过执行 PSUBSCRIBE * 命令可以接收所有事件信息。

> @ 字符之后的内容用于指定主服务器， 这些内容是可选的， 它们仅在 @ 字符之前的内容指定的实例不是主服务器时使用。
>
> - +reset-master ：主服务器已被重置。
> - +slave ：一个新的从服务器已经被 Sentinel 识别并关联。
> - +failover-state-reconf-slaves ：故障转移状态切换到了 reconf-slaves 状态。
> - +failover-detected ：另一个 Sentinel 开始了一次故障转移操作，或者一个从服务器转换成了主服务器。
> - +slave-reconf-sent ：领头（leader）的 Sentinel 向实例发送了 [SLAVEOF](/commands/slaveof.html) 命令，为实例设置新的主服务器。
> - +slave-reconf-inprog ：实例正在将自己设置为指定主服务器的从服务器，但相应的同步过程仍未完成。
> - +slave-reconf-done ：从服务器已经成功完成对新主服务器的同步。
> - -dup-sentinel ：对给定主服务器进行监视的一个或多个 Sentinel 已经因为重复出现而被移除 —— 当 Sentinel 实例重启的时候，就会出现这种情况。
> - +sentinel ：一个监视给定主服务器的新 Sentinel 已经被识别并添加。
> - +sdown ：给定的实例现在处于主观下线状态。
> - -sdown ：给定的实例已经不再处于主观下线状态。
> - +odown ：给定的实例现在处于客观下线状态。
> - -odown ：给定的实例已经不再处于客观下线状态。
> - +new-epoch ：当前的纪元（epoch）已经被更新。
> - +try-failover ：一个新的故障迁移操作正在执行中，等待被大多数 Sentinel 选中（waiting to be elected by the majority）。
> - +elected-leader ：赢得指定纪元的选举，可以进行故障迁移操作了。
> - +failover-state-select-slave ：故障转移操作现在处于 select-slave 状态 —— Sentinel 正在寻找可以升级为主服务器的从服务器。
> - no-good-slave ：Sentinel 操作未能找到适合进行升级的从服务器。Sentinel 会在一段时间之后再次尝试寻找合适的从服务器来进行升级，又或者直接放弃执行故障转移操作。
> - selected-slave ：Sentinel 顺利找到适合进行升级的从服务器。
> - failover-state-send-slaveof-noone ：Sentinel 正在将指定的从服务器升级为主服务器，等待升级功能完成。
> - failover-end-for-timeout ：故障转移因为超时而中止，不过最终所有从服务器都会开始复制新的主服务器（slaves will eventually be configured to replicate with the new master anyway）。
> - failover-end ：故障转移操作顺利完成。所有从服务器都开始复制新的主服务器了。
> - +switch-master ：配置变更，主服务器的 IP 和地址已经改变。 这是绝大多数外部用户都关心的信息。
> - +tilt ：进入 tilt 模式。
> - -tilt ：退出 tilt 模式。

​	

## 2. 实操

​	模拟主节点下线，sentinel选举过程

### 1）正常状态：

搭建redis主从集群

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210428173633.png" style="zoom:50%;" />

```bash
[root@localhost ~]# redis-cli
127.0.0.1:6379> role
1) "master"
2) (integer) 21919
3) 1) 1) "127.0.0.1"
      2) "6380"
      3) "21905"
   2) 1) "127.0.0.1"
      2) "6381"
      3) "21919"
```

6379主节点哨兵

```bash
29686:X 29 Apr 2021 11:35:03.333 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
29686:X 29 Apr 2021 11:35:03.335 # Sentinel ID is 750b231a6bc075f3c26e9778ad571a2c871014a5
29686:X 29 Apr 2021 11:35:03.335 # +monitor master mymaster 127.0.0.1 6379 quorum 2
29686:X 29 Apr 2021 11:35:03.337 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
29686:X 29 Apr 2021 11:35:03.338 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
29686:X 29 Apr 2021 11:36:14.644 * +sentinel sentinel b56fe20283d8772e11d6abbccc23b74ee36336c9 127.0.0.1 26380 @ mymaster 127.0.0.1 6379
29686:X 29 Apr 2021 11:36:46.501 * +sentinel sentinel 80c8279952d73d0aae6254a024c05facdbdf7efb 127.0.0.1 26381 @ mymaster 127.0.0.1 6379
```

6380从节点哨兵

```bash

29783:X 29 Apr 2021 11:36:12.596 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
29783:X 29 Apr 2021 11:36:12.598 # Sentinel ID is b56fe20283d8772e11d6abbccc23b74ee36336c9
29783:X 29 Apr 2021 11:36:12.598 # +monitor master mymaster 127.0.0.1 6380 quorum 2
29783:X 29 Apr 2021 11:36:12.623 * +sentinel sentinel 750b231a6bc075f3c26e9778ad571a2c871014a5 127.0.0.1 26379 @ mymaster 127.0.0.1 6380
29783:X 29 Apr 2021 11:37:02.646 # +sdown master mymaster 127.0.0.1 6380
```

6381从节点哨兵

```bash
29857:X 29 Apr 2021 11:36:44.502 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
29857:X 29 Apr 2021 11:36:44.504 # Sentinel ID is 80c8279952d73d0aae6254a024c05facdbdf7efb
29857:X 29 Apr 2021 11:36:44.504 # +monitor master mymaster 127.0.0.1 6381 quorum 2
29857:X 29 Apr 2021 11:36:45.264 * +sentinel sentinel 750b231a6bc075f3c26e9778ad571a2c871014a5 127.0.0.1 26379 @ mymaster 127.0.0.1 6381


29857:X 29 Apr 2021 11:37:34.562 # +sdown master mymaster 127.0.0.1 6381
```

### 2）模拟主节点下线状态

```bash
redis-cli -p 6379 DEBUG sleep 80
```

- 哨兵日志：

```bash
29686:X 29 Apr 2021 11:55:10.883 # +new-epoch 1
29686:X 29 Apr 2021 11:55:10.884 # +vote-for-leader cb1dd7eb638b1c9699a9be3458e36224f47a7a02 1
29686:X 29 Apr 2021 11:55:11.233 # +sdown master mymaster 127.0.0.1 6379
29686:X 29 Apr 2021 11:55:11.333 # +odown master mymaster 127.0.0.1 6379 #quorum 3/2
29686:X 29 Apr 2021 11:55:11.333 # Next failover delay: I will not start a failover before Thu Apr 29 12:01:10 2021
29686:X 29 Apr 2021 11:55:11.997 # +config-update-from sentinel cb1dd7eb638b1c9699a9be3458e36224f47a7a02 127.0.0.1 26381 @ mymaster 127.0.0.1 6379
29686:X 29 Apr 2021 11:55:11.997 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6381
29686:X 29 Apr 2021 11:55:11.997 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6381
29686:X 29 Apr 2021 11:55:11.998 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
29686:X 29 Apr 2021 11:55:42.005 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
29686:X 29 Apr 2021 11:56:00.871 # -sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
```

- 重新进行选举，slave6381成为新的主节点

```bash
127.0.0.1:26379> SENTINEL get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "6381"
```

- 睡眠时间到，6379节点重新上线，发现角色已经被切换

  哨兵日志：

  ```bash
  29686:X 29 Apr 2021 11:56:10.803 * +convert-to-slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
  ```

  6379节点ROLE输出：

	```bash
	[root@localhost ~]# redis-cli -p 6379
	127.0.0.1:6379> ROLE
	1) "slave"
	2) "127.0.0.1"
	3) (integer) 6381
	4) "connected"
	5) (integer) 180039
	```

- 查看哨兵配置文件

  ```bash
  port 26379
  sentinel myid 750b231a6bc075f3c26e9778ad571a2c871014a5
  # Generated by CONFIG REWRITE
  protected-mode no
  user default on nopass ~* +@all
  dir "/etc/redis"
  sentinel deny-scripts-reconfig yes
  sentinel monitor mymaster 127.0.0.1 6381 2
  sentinel config-epoch mymaster 3
  sentinel leader-epoch mymaster 3
  sentinel known-replica mymaster 127.0.0.1 6379
  sentinel known-replica mymaster 127.0.0.1 6380
  sentinel known-sentinel mymaster 127.0.0.1 26380 67567fae74db24ca01ac13b2ba9f51c7960cf7cd
  sentinel known-sentinel mymaster 127.0.0.1 0 80c8279952d73d0aae6254a024c05facdbdf7efb
  sentinel known-sentinel mymaster 127.0.0.1 26381 cb1dd7eb638b1c9699a9be3458e36224f47a7a02
  sentinel known-sentinel mymaster 127.0.0.1 0 b56fe20283d8772e11d6abbccc23b74ee36336c9
  sentinel current-epoch 3
  ```

  发现配置文件守护的主节点已经变为了6381，所以下次6381节点出现问题，也能及时进行监控。

- 查看sentinel发布订阅消息正常

  ```bash
  [root@localhost ~]# redis-cli -p 6379
  127.0.0.1:6379> PSUBSCRIBE *
  Reading messages... (press Ctrl-C to quit)
  1) "psubscribe"
  2) "*"
  3) (integer) 1
  1) "pmessage"
  2) "*"
  3) "__sentinel__:hello"
  4) "127.0.0.1,26381,cb1dd7eb638b1c9699a9be3458e36224f47a7a02,3,mymaster,127.0.0.1,6381,3"
  1) "pmessage"
  2) "*"
  3) "__sentinel__:hello"
  4) "127.0.0.1,26380,67567fae74db24ca01ac13b2ba9f51c7960cf7cd,3,mymaster,127.0.0.1,6381,3"
  1) "pmessage"
  2) "*"
  3) "__sentinel__:hello"
  4) "127.0.0.1,26379,750b231a6bc075f3c26e9778ad571a2c871014a5,3,mymaster,127.0.0.1,6381,3"
  1) "pmessage"
  2) "*"
  ```

  


