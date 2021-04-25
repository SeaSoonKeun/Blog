# Redis持久化-RDB-AOF


# Redis持久化-RDB-AOF

我们都知道Redis所有的数据都存在内存中，这与传统的MySQL，Oracle等关系型数据库直接将内容保存到硬盘中相比，内存数据库的读写效率比传统数据库要快的多（内存的读写效率远远大于硬盘的读写效率）。但是保存在内存中也随之带来了一个缺点，一旦断电或者宕机，那么内存数据库中的数据将会全部丢失。

为了解决这个缺点，Redis提供了将内存数据持久化到硬盘，以及用持久化文件来恢复数据库数据的功能，从内存当中同步到硬盘上，这个过程叫做**持久化过程**。

​	存储层：

1，快照/副本 RDB

2，日志	AOF

## 1. RDB 时间点快照（point-in-time snapshot）

redis database

RDB持久化就是把当前Redis数据库中的内存数据保存到硬盘的过程，RDB方式是Redis默认支持的。

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/RDB%E6%8C%81%E4%B9%85%E5%8C%96-fork.jpg" style="zoom:50%;" />



### 触发时机

RDB持久化的触发方式有两种，第一种是自动触发，另外一种是手动触发。

#### 1) 自动触发

通过查看redis.conf里面的SNAPSHOTTING内容可知：

```bash
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb

# Remove RDB files used by replication in instances without persistence
                                                                                                                                     342,1         16%
# in order to load them for the initial synchronization, should be deleted
# ASAP. Note that this option ONLY WORKS in instances that have both AOF
# and RDB persistence disabled, otherwise is completely ignored.
#
# An alternative (and sometimes better) way to obtain the same effect is
# to use diskless replication on both master and replicas instances. However
# in the case of replicas, diskless is not always an option.
rdb-del-sync-files no

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /var/lib/redis/6379

```

1、save：这是**默认触发**Redis的RDB持久化的条件，如果你只是用Redis的缓存功能，不需要持久化，那么你可以注释掉所有save的默认配置来停用保存功能。可以直接一个空字符串来实现停用：save " "。

默认配置：
```
save 900 1：    表示900 秒内如果至少有 1 个 key 的值变化，则保存
save 300 10：   表示300 秒内如果至少有 10 个 key 的值变化，则保存
save 60  10000：表示60  秒内如果至少有 10000 个 key 的值变化，则保存
```
　　2、stop-writes-on-bgsave-error ：默认值为yes。当启用了RDB且最后一次后台保存数据失败，Redis是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难（disaster）发生了。如果Redis重启了，那么又可以重新开始接收数据了

　　3、rdbcompression ；默认值是yes。对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。

　　4、rdbchecksum ：默认值是yes。在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。

　　5、dbfilename ：设置快照的文件名，默认是 dump.rdb

　　6、dir：设置快照文件的存放路径，这个配置项一定是个目录，而不能是文件名。默认是和当前配置文件保存在同一目录。

#### 2) 手动触发

通过命令save或者是bgsave

save：save时只管保存，其它不管，**全部阻塞**。

bgsave：Redis会在后台**异步进行**快照操作，**快照同时还可以响应客户端请求**。可以通过lastsave命令获取最后一次成功执行快照的时间。

## 2.AOF**只追加文件**


