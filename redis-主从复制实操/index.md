# Redis-主从复制实操

# Redis-主从复制实操（手动触发）

## 1. 正常环境：

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210428173633.png" style="zoom:50%;" />

- 环境配置：

  6380请求6379

```bash
127.0.0.1:6380> REPLICAOF 127.0.0.1 6379
OK
```

6380前台日志发送连接请求 

```bash
4299:S 28 Apr 2021 05:59:54.271 * REPLICAOF 127.0.0.1:6379 enabled (user request from 'id=5 addr=127.0.0.1:60264 fd=7 name= age=10 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=44 qbuf-free=32724 obl=0 oll=0 omem=0 events=r cmd=replicaof user=default')
4299:S 28 Apr 2021 05:59:55.148 * Connecting to MASTER 127.0.0.1:6379
4299:S 28 Apr 2021 05:59:55.149 * MASTER <-> REPLICA sync started
4299:S 28 Apr 2021 05:59:55.149 * Non blocking connect for SYNC fired the event.
4299:S 28 Apr 2021 05:59:55.150 * Master replied to PING, replication can continue...
4299:S 28 Apr 2021 05:59:55.150 * Trying a partial resynchronization (request a41351c70395134df2cdaa9848e7b2a49135376f:1).
4299:S 28 Apr 2021 05:59:55.151 * Full resync from master: df1446eb8808e34a75862c92f4e92dc2e222caf8:0
4299:S 28 Apr 2021 05:59:55.151 * Discarding previously cached master state.
4299:S 28 Apr 2021 05:59:55.244 * MASTER <-> REPLICA sync: receiving 175 bytes from master to disk
4299:S 28 Apr 2021 05:59:55.244 * MASTER <-> REPLICA sync: Flushing old data
4299:S 28 Apr 2021 05:59:55.244 * MASTER <-> REPLICA sync: Loading DB in memory
4299:S 28 Apr 2021 05:59:55.244 * Loading RDB produced by version 6.0.6
4299:S 28 Apr 2021 05:59:55.244 * RDB age 0 seconds
4299:S 28 Apr 2021 05:59:55.244 * RDB memory usage when created 1.83 Mb
4299:S 28 Apr 2021 05:59:55.244 * MASTER <-> REPLICA sync: Finished with success
```

```bash
4299:S 28 Apr 2021 06:04:15.030 * 1 changes in 900 seconds. Saving...
4299:S 28 Apr 2021 06:04:15.031 * Background saving started by pid 5197
5197:C 28 Apr 2021 06:04:15.034 * DB saved on disk
5197:C 28 Apr 2021 06:04:15.035 * RDB: 0 MB of memory used by copy-on-write
4299:S 28 Apr 2021 06:04:15.131 * Background saving terminated with success
```

6379前台日志收到连接请求建立连接

```bash
4268:M 28 Apr 2021 05:59:55.150 * Replica 127.0.0.1:6380 asks for synchronization
4268:M 28 Apr 2021 05:59:55.150 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for 'a41351c70395134df2cdaa9848e7b2a49135376f', my replication IDs are '6b283944b97758ddf1f9aea9b3182c9121649d8e' and '0000000000000000000000000000000000000000')
4268:M 28 Apr 2021 05:59:55.150 * Replication backlog created, my new replication IDs are 'df1446eb8808e34a75862c92f4e92dc2e222caf8' and '0000000000000000000000000000000000000000'
4268:M 28 Apr 2021 05:59:55.150 * Starting BGSAVE for SYNC with target: disk
4268:M 28 Apr 2021 05:59:55.151 * Background saving started by pid 4929
4929:C 28 Apr 2021 05:59:55.154 * DB saved on disk
4929:C 28 Apr 2021 05:59:55.155 * RDB: 0 MB of memory used by copy-on-write
4268:M 28 Apr 2021 05:59:55.243 * Background saving terminated with success
4268:M 28 Apr 2021 05:59:55.244 * Synchronization with replica 127.0.0.1:6380 succeeded
```

```bash
4268:M 28 Apr 2021 06:14:56.052 * 1 changes in 900 seconds. Saving...
4268:M 28 Apr 2021 06:14:56.053 * Background saving started by pid 5751
5751:C 28 Apr 2021 06:14:56.055 * DB saved on disk
5751:C 28 Apr 2021 06:14:56.056 * RDB: 0 MB of memory used by copy-on-write
4268:M 28 Apr 2021 06:14:56.153 * Background saving terminated with success
```

注意：**只要从节点开启了AOF主从之间就是全量同步**DB文件

6381前台日志收到连接请求建立连接同样，省略



## 2. 手动关闭主，模拟主节点挂掉的情况。

- 从节点报错，连接失败。

```bash
26050:S 28 Apr 2021 13:47:28.731 * Connecting to MASTER 127.0.0.1:6379
26050:S 28 Apr 2021 13:47:28.731 * MASTER <-> REPLICA sync started
26050:S 28 Apr 2021 13:47:28.731 # Error condition on socket for SYNC: Operation now in progress
```

- 因为是人工指定主从的方式，**不会进行高可用切换。**
- 从节点**手动启动**主节点选举。

```bash
127.0.0.1:6380> REPLICAOF no one
OK
```

此时6381 redis进程并不会与6380建立连接，还是一直在与6379请求连接。**需要人工维护**主的故障问题，由此诞生了sentinel哨兵。


