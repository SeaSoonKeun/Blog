# Redis持久化-RDB/AOF


# Redis持久化-RDB/AOF

我们都知道Redis所有的数据都存在内存中，这与传统的MySQL，Oracle等关系型数据库直接将内容保存到硬盘中相比，内存数据库的读写效率比传统数据库要快的多（内存的读写效率远远大于硬盘的读写效率）。但是保存在内存中也随之带来了一个缺点，一旦断电或者宕机，那么内存数据库中的数据将会全部丢失。

为了解决这个缺点，Redis提供了将**内存数据持久化到硬盘**，以及用持久化文件来恢复数据库数据的功能，从内存当中同步到硬盘上，这个过程叫做**持久化过程**。

​	存储层：

1，快照/副本 RDB

2，日志	AOF



## 1. RDB 时间点快照（point-in-time snapshot）

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210426231815.png" style="zoom:50%;" />

### 1. **redis database**时点性

RDB持久化就是把当前Redis数据库中的内存数据保存到硬盘的过程，RDB方式是Redis默认支持的。

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/RDB%E6%8C%81%E4%B9%85%E5%8C%96-fork.jpg" style="zoom:50%;" />



每执行一次修改，会通过fork执行一次写时复制(COW)

实现异步落盘

### 2. 触发时机

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
通过save命令执行bgsave操作
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

save <seconds> <changes>

```
save 900 1：    表示900 秒内如果至少有 1 个 key 的值变化，则保存
save 300 10：   表示300 秒内如果至少有 10 个 key 的值变化，则保存
save 60  10000：表示60  秒内如果至少有 10000 个 key 的值变化，则保存
```
　　2、stop-writes-on-bgsave-error ：默认值为yes。当启用了RDB且最后一次后台保存数据失败，Redis是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难（disaster）发生了。如果Redis重启了，那么又可以重新开始接收数据了

　　3、rdbcompression ；默认值是yes。对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。

　　4、rdbchecksum ：默认值是yes。在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。

　　5、dbfilename ：设置快照的**文件名**，默认是 dump.rdb

　　6、dir：设置快照文件的**存放路径**，这个配置项一定是个目录，而不能是文件名。默认是和当前配置文件保存在同一目录。

#### 2) 手动触发

通过命令save或者是bgsave

save：save时只管保存，其它不管，**全部阻塞**。关机维护的时候

bgsave：Redis会在后台**异步进行**快照操作，**快照同时还可以响应客户端请求**。可以通过lastsave命令获取最后一次成功执行快照的时间。

### 3. 弊端

1. 不支持拉链，每次都是全量dump.rdb。
2. 丢失数据相对多一些。时点间的数据窗口数据容易丢失。

### 4. 优点

类似于java中的序列化，**恢复的速度相对快**。



## 2. AOF **只追加文件**

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/Redis_AOF.jpg)

### 1）配置文件

```bash
############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"

# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no

# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no

# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes

# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
aof-use-rdb-preamble yes
```

#### 精简配置文件：

```bash
appendonly no
appendfilename "appendonly.aof"
# 配置 Redis 多久才将数据 fsync 到磁盘一次
# 结合具体业务场景调整aof写周期
# 每次有新命令追加到 AOF 文件时就执行一次 fsync ：非常慢，也非常安全。
# 每秒 fsync 一次：足够快（和使用 RDB 持久化差不多），并且在故障时只会丢失 1 秒钟的数据。
# 从不 fsync ：将数据交给操作系统来处理。更快，也更不安全的选择。会造成内核buffer中的数据丢失。
# 推荐（并且也是默认）的措施为每秒 fsync 一次， 这种 fsync 策略可以兼顾速度和安全性。
# appendfsync always
appendfsync everysec
# appendfsync no

# man 详解fsync()
# NAME
#       fsync, fdatasync - synchronize a file's in-core state with storage device
# 当AOF fsync policy被设置为always或者everysecond时，会对磁盘产生较大的IO压力，Redis会阻塞很长时间在fsync()调用上。为了缓解这个压力，下面的参数会在BGSAVE或者BGREWRITEAOF执行时，阻止主进程的其他fsync()调用。默认为关闭状态，如果有严重的延迟问题，需要开启。
no-appendfsync-on-rewrite no

# 自动重写aof文件大小百分比
auto-aof-rewrite-percentage 100
# 自动重写aof文件大小
auto-aof-rewrite-min-size 64mb
# redis出现异常时
aof-load-truncated yes
# 开启4.0版本后的，rdb + aof 融合
aof-use-rdb-preamble yes
```

### 实操比较两种AOF重写方式：

### 1）重写 （4.0版本之前）

修改配置文件

```bash
# 关闭后台守护，方便看前台日志
daemonize no
# 注释掉日志文件输出，直接在前台输出
# logfile /var/log/redis_6379.log
# 只开启aof
appendonly yes
# 关闭默认开启的混合体模式
aof-use-rdb-preamble no
```

清理老的文件

```bash
cd /var/lib/redis/6379
rm -rf dump.rdb appendonly.aof
```

启动服务

```bash
redis-server /etc/redis/6379_aof_rewrite.conf
```

现象

```bash
# 未执行操作时，aof文件为空
[root@localhost 6379]# ll appendonly.aof
-rw-r--r--. 1 root root 0 4月  26 14:24 appendonly.aof

# 设置一个key
127.0.0.1:6379> set k1 aaa
OK

# 查看appendonly.aof文件内容，为写操作指令
*2
$6
SELECT
$1
0
*3
$3
set
$2
k1
$3
aaa

# 手动调用rdb bgsave进行写rdb文件
127.0.0.1:6379> bgsave
Background saving started
# 前台日志提示，后台子进程进行写时复制执行
15113:M 26 Apr 2021 14:28:50.513 * Background saving started by pid 15352
15352:C 26 Apr 2021 14:28:50.520 * DB saved on disk
15352:C 26 Apr 2021 14:28:50.521 * RDB: 8 MB of memory used by copy-on-write
15113:M 26 Apr 2021 14:28:50.620 * Background saving terminated with success
# 生成了rdb文件
[root@localhost 6379]# ll
总用量 8
-rw-r--r--. 1 root root  53 4月  26 14:25 appendonly.aof
-rw-r--r--. 1 root root 105 4月  26 14:28 dump.rdb
# rdb文件内容为字节码文件，开头为REDIS。
REDIS0009ú      redis-ver^E6.0.6ú
redis-bitsÀ@ú^EctimeÂb^F<87>`ú^Hused-memÂ<80>3^M^@ú^Laof-preambleÀ^@þ^@û^A^@^@^Bk1^Caaaÿ<97><8c>|^S<80>¾V_
# 查看rdb文件
[root@localhost 6379]# redis-check-rdb dump.rdb
[offset 0] Checking RDB file dump.rdb
[offset 26] AUX FIELD redis-ver = '6.0.6'
[offset 40] AUX FIELD redis-bits = '64'
[offset 52] AUX FIELD ctime = '1619461730'
[offset 67] AUX FIELD used-mem = '865152'
[offset 83] AUX FIELD aof-preamble = '0'
[offset 85] Selecting DB ID 0
[offset 105] Checksum OK
[offset 105] \o/ RDB looks OK! \o/
[info] 1 keys read
[info] 0 expires
[info] 0 already expired

# 对同一个key执行多次写操作
127.0.0.1:6379> set k1 aaa
OK
127.0.0.1:6379> bgsave
Background saving started
127.0.0.1:6379> set k1 bbb
OK
127.0.0.1:6379> set k1 ccc
OK
127.0.0.1:6379> set k1 ddd
OK
# aof文件
# 其实内存中只有一个值有效，内存中保存了大量无效数据
*2
$6
SELECT
$1
0
*3
$3
set
$2
k1
$3
aaa
*3
$3
set
$2
k1
$3
bbb
*3
$3
set
$2
k1
$3
ccc
*3
$3
set
$2
k1
$3
ddd
# 手动调用重写aof命令
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
# 前台日志
15113:M 26 Apr 2021 14:36:38.629 * Background append only file rewriting started by pid 15756
15113:M 26 Apr 2021 14:36:38.660 * AOF rewrite child asks to stop sending diffs.
15756:C 26 Apr 2021 14:36:38.660 * Parent agreed to stop sending diffs. Finalizing AOF...
15756:C 26 Apr 2021 14:36:38.660 * Concatenating 0.00 MB of AOF diff received from parent.
15756:C 26 Apr 2021 14:36:38.660 * SYNC append only file rewrite performed
15756:C 26 Apr 2021 14:36:38.661 * AOF rewrite: 8 MB of memory used by copy-on-write
15113:M 26 Apr 2021 14:36:38.739 * Background AOF rewrite terminated with success
15113:M 26 Apr 2021 14:36:38.739 * Residual parent diff successfully flushed to the rewritten AOF (0.00 MB)
15113:M 26 Apr 2021 14:36:38.739 * Background AOF rewrite finished successfully
# 重新查看aof文件，已经删除了无效数据，重新写入。
[root@localhost redis]# cd 6379/
*2
$6
SELECT
$1
0
*3
$3
SET
$2
k1
$3
ddd
```



### 2）重写 + RDB 混合体（4.0版本之后）

```bash
# 恢复配置
appendonly yes
aof-use-rdb-preamble yes

# 启动服务
# 查看目录下默认就已经生成aof文件
[root@localhost 6379]# pwd
/var/lib/redis/6379
appendonly.aof

# 添加key操作，目录下生成aof文件记录
*2
$6
SELECT
$1
0
*3
$3
set
$2
k1
$3
111
*3
$3
set
$2
k2
$3
aaa
*3
$3
set
$2
k3
$3
xcg

# 重写之后才能变为混合体,生成redis开头的aof文件
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
[root@localhost 6379]# vim appendonly.aof
REDIS0009ú      redis-ver^E6.0.6ú
redis-bitsÀ@ú^EctimeÂNw<87>`ú^Hused-memÂØ3^M^@ú^Laof-preambleÀ^Aþ^@û^C^@^@^Bk2^Caaa^@^Bk3^Cxcg^@^Bk1^Bk1ÿ<9b>«Á.<9e>^?,^?

# 增量明文日志，混合体
REDIS0009ú      redis-ver^E6.0.6ú
redis-bitsÀ@ú^EctimeÂNw<87>`ú^Hused-memÂØ3^M^@ú^Laof-preambleÀ^Aþ^@û^C^@^@^Bk2^Caaa^@^Bk3^Cxcg^@^Bk1^Bk1ÿ<9b>«Á.<9e>^?,^?*2^M
$6^M
SELECT^M
$1^M
0^M
*3^M
$3^M
set^M
$2^M
k1^M
$1^M
a^M
*3^M
$3^M
set^M
$2^M
k2^M
$1^M
b^M
~
```



## 3. 官网对两种持久化方式优缺点分析

### **RDB 的优点:**

- RDB是一种表示某个即时点的Redis数据的**紧凑文件**。RDB文件适合用于备份。例如，你可能想要每小时归档最近24小时的RDB文件，每天保存近30天的RDB快照。这允许你很容易的恢复不同版本的数据集以容灾。
- RDB非常适合于灾难恢复，作为一个紧凑的单一文件，可以被传输到远程的数据中心，或者是Amazon S3(可能得加密)。
- **RDB最大化了Redis的性能**，因为Redis父进程持久化时唯一需要做的是启动(fork)一个子进程，由子进程完成所有剩余工作。父进程实例不需要执行像磁盘IO这样的操作。
- RDB在重启保存了大数据集的实例时比AOF要快。



### **RDB 的缺点**

- 当你需要在Redis停止工作(例如停电)时最小化数据丢失，RDB可能不太好。你可以配置不同的保存点(save point)来保存RDB文件(例如，至少5分钟和对数据集100次写之后，但是你可以有多个保存点)。然而，你通常每隔5分钟或更久创建一个RDB快照，所以一旦Redis因为任何原因没有正确关闭而停止工作，你就得做好最近几分钟数据丢失的准备了。
- RDB需要经常调用fork()子进程来持久化到磁盘。如果数据集很大的话，fork()比较耗时，结果就是，当数据集非常大并且CPU性能不够强大的话，Redis会停止服务客户端几毫秒甚至一秒。AOF也需要fork()，但是你可以调整多久频率重写日志而不会有损(trade-off)持久性(durability)。



### **AOF 的优点:**

- 使用AOF Redis会更具有可持久性(durable)：你可以有很多不同的fsync策略：没有fsync，每秒fsync，每次请求时fsync。使用默认的每秒fsync策略，写性能也仍然很不错(fsync是由后台线程完成的，主线程继续努力地执行写请求)，即便你也就仅仅只损失一秒钟的写数据。
- AOF日志是一个追加文件，所以不需要定位，在断电时也没有损坏问题。即使由于某种原因文件末尾是一个写到一半的命令(磁盘满或者其他原因),redis-check-aof工具也可以很轻易的修复。
- 当AOF文件变得很大时，Redis会自动在后台进行重写。重写是绝对安全的，因为Redis继续往旧的文件中追加，使用创建当前数据集所需的最小操作集合来创建一个全新的文件，一旦第二个文件创建完毕，Redis就会切换这两个文件，并开始往新文件追加。
- AOF文件里面包含一个接一个的操作，以易于理解和解析的格式存储。你也可以轻易的导出一个AOF文件。例如，即使你不小心错误地使用FLUSHALL命令清空一切，如果此时并没有执行重写，你仍然可以保存你的数据集，你只要停止服务器，删除最后一条命令，然后重启Redis就可以。



### **AOF 的缺点:**

- 对同样的数据集，AOF文件通常要**大**于等价的RDB文件。
- AOF可能比RDB**慢**，这取决于准确的fsync策略。通常fsync设置为每秒一次的话性能仍然很高，如果关闭fsync，即使在很高的负载下也和RDB一样的快。不过，即使在很大的写负载情况下，RDB还是能提供能好的最大延迟保证。
- 在过去，我们经历了一些针对特殊命令(例如，像BRPOPLPUSH这样的阻塞命令)的罕见bug，导致在数据加载时无法恢复到保存时的样子。这些bug很罕见，我们也在测试套件中进行了测试，自动随机创造复杂的数据集，然后加载它们以检查一切是否正常，但是，这类bug几乎不可能出现在RDB持久化中。为了说得更清楚一点：Redis AOF是通过递增地更新一个已经存在的状态，像MySQL或者MongoDB一样，而RDB快照是一次又一次地从头开始创造一切，概念上更健壮。但是，1)要注意Redis每次重写AOF时都是以当前数据集中的真实数据从头开始，相对于一直追加的AOF文件(或者一次重写读取老的AOF文件而不是读内存中的数据)对bug的免疫力更强。2)我们还没有收到一份用户在真实世界中检测到崩溃的报告。

## 4. 备份Redis数据库建议

- 创建一个**定时任务(cron job)**，每隔一个小时**创建一个RDB快照到一个目录**，每天的快照放在不同目录。
- 每次定时脚本运行时，务必使用find命令来**删除旧的快照**：例如，你可以保存最近48小时内的每小时快照，一到两个月的内的每天快照。注意命名快照时加上日期时间信息。
- 至少每天一次将你的RDB快照传输到你的**数据中心之外**，或者至少传输到运行你的Redis实例的物理机之外。
