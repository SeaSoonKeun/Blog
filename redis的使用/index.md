# Redis的使用

# Redis的使用

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/Redis_String%E7%B1%BB%E5%9E%8B%E6%95%B4%E4%BD%93%E5%9B%BE.jpg)

`redis-cli` 进入客户端

```bash
[root@localhost ~]# redis-cli --help & redis-cli -h
redis-cli 6.0.6

Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -s <socket>        Server socket (overrides hostname and port).
  -a <password>      Password to use when connecting to the server.
                     You can also use the REDISCLI_AUTH environment
                     variable to pass this password more safely
                     (if both are used, this argument takes predecence).
  --user <username>  Used to send ACL style 'AUTH username pass'. Needs -a.
  --pass <password>  Alias of -a for consistency with the new --user option.
  --askpass          Force user to input password with mask from STDIN.
                     If this argument is used, '-a' and REDISCLI_AUTH
                     environment variable will be ignored.
  -u <uri>           Server URI.
  -r <repeat>        Execute specified command N times.
  -i <interval>      When -r is used, waits <interval> seconds per command.
                     It is possible to specify sub-second times like -i 0.1.
  -n <db>            Database number.
  -3                 Start session in RESP3 protocol mode.
  -x                 Read last argument from STDIN.
  -d <delimiter>     Multi-bulk delimiter in for raw formatting (default: \n).
  -c                 Enable cluster mode (follow -ASK and -MOVED redirections).
  --raw              Use raw formatting for replies (default when STDOUT is
                     not a tty).
  --no-raw           Force formatted output even when STDOUT is not a tty.
  --csv              Output in CSV format.
  --stat             Print rolling stats about server: mem, clients, ...
  --latency          Enter a special mode continuously sampling latency.
                     If you use this mode in an interactive session it runs
                     forever displaying real-time stats. Otherwise if --raw or
                     --csv is specified, or if you redirect the output to a non
                     TTY, it samples the latency for 1 second (you can use
                     -i to change the interval), then produces a single output
                     and exits.
  --latency-history  Like --latency but tracking latency changes over time.
                     Default time interval is 15 sec. Change it using -i.
  --latency-dist     Shows latency as a spectrum, requires xterm 256 colors.
                     Default time interval is 1 sec. Change it using -i.
  --lru-test <keys>  Simulate a cache workload with an 80-20 distribution.
  --replica          Simulate a replica showing commands received from the master.
  --rdb <filename>   Transfer an RDB dump from remote server to local file.
  --pipe             Transfer raw Redis protocol from stdin to server.
  --pipe-timeout <n> In --pipe mode, abort with error if after sending all data.
                     no reply is received within <n> seconds.
                     Default timeout: 30. Use 0 to wait forever.
  --bigkeys          Sample Redis keys looking for keys with many elements (complexity).
  --memkeys          Sample Redis keys looking for keys consuming a lot of memory.
  --memkeys-samples <n> Sample Redis keys looking for keys consuming a lot of memory.
                     And define number of key elements to sample
  --hotkeys          Sample Redis keys looking for hot keys.
                     only works when maxmemory-policy is *lfu.
  --scan             List all keys using the SCAN command.
  --pattern <pat>    Useful with --scan to specify a SCAN pattern.
  --intrinsic-latency <sec> Run a test to measure intrinsic system latency.
                     The test will run for the specified amount of seconds.
  --eval <file>      Send an EVAL command using the Lua script at <file>.
  --ldb              Used with --eval enable the Redis Lua debugger.
  --ldb-sync-mode    Like --ldb but uses the synchronous Lua debugger, in
                     this mode the server is blocked and script changes are
                     not rolled back from the server memory.
  --cluster <command> [args...] [opts...]
                     Cluster Manager command and arguments (see below).
  --verbose          Verbose mode.
  --no-auth-warning  Don't show warning message when using password on command
                     line interface.
  --help             Output this help and exit.
  --version          Output version and exit.

Cluster Manager Commands:
  Use --cluster help to list all available cluster manager commands.

Examples:
  cat /etc/passwd | redis-cli -x set mypasswd
  redis-cli get mypasswd
  redis-cli -r 100 lpush mylist x
  redis-cli -r 100 -i 1 info | grep used_memory_human:
  redis-cli --eval myscript.lua key1 key2 , arg1 arg2 arg3
  redis-cli --scan --pattern '*:12345*'

  (Note: when using --eval the comma separates KEYS[] from ARGV[] items)

When no command is given, redis-cli starts in interactive mode.
Type "help" in interactive mode for information on available commands
and settings.
```

常用：

`-h`

`-b`

`-n`

## 库表结构

- 库：Redis 默认16个库 
  - `-n`指定连接的库
  - 默认为`0`号库
  - select x 进行切换库

## help使用

```shell
127.0.0.1:6379> help
redis-cli 6.0.6
To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit

To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
```

## help @命令组

- 例如help @generic

  ```bash
  127.0.0.1:6379> help @generic
  
    DEL key [key ...]
    summary: Delete a key
    since: 1.0.0
  
    DUMP key
    summary: Return a serialized version of the value stored at the specified key.
    since: 2.6.0
  
    EXISTS key [key ...]
    summary: Determine if a key exists
    since: 1.0.0
  
    EXPIRE key seconds
    summary: Set a key's time to live in seconds
    since: 1.0.0
  
    EXPIREAT key timestamp
    summary: Set the expiration for a key as a UNIX timestamp
    since: 1.2.0
  
    KEYS pattern
    summary: Find all keys matching the given pattern
    since: 1.0.0
  
    MIGRATE host port key| destination-db timeout [COPY] [REPLACE] [AUTH password] [KEYS key]
    summary: Atomically transfer a key from a Redis instance to another one.
    since: 2.6.0
  
    MOVE key db
    summary: Move a key to another database
    since: 1.0.0
  
    OBJECT subcommand [arguments [arguments ...]]
    summary: Inspect the internals of Redis objects
    since: 2.2.3
  ```

  

- help tab可以补全

- keys * 显示所有key

- FLUSHDB 清库，测试环境下使用

  ```shell
  127.0.0.1:6379>  keys *
  1) "k380"
  2) "k380:1"
  127.0.0.1:6379> FlushDB
  OK
  127.0.0.1:6379>  keys *
  (empty array)
  127.0.0.1:6379>
  ```

## 5种基本类型

### `1. help @string`

#### Set 

> #### 设置值

```bash
# 查看set帮助
127.0.0.1:6379> help set

  SET key value [EX seconds|PX milliseconds] [NX|XX] [KEEPTTL]
  summary: Set the string value of a key
  since: 1.0.0
  group: string
```

- NX -> not x -> not exist 不存在的时候才去设置

  - 使用场景：分布式锁。谁先成功谁就拿到锁了

  - ```
    127.0.0.1:6379> set k1 hello nx
    OK
    127.0.0.1:6379> set k1 world nx
    (nil)
    127.0.0.1:6379> get k1
    "hello"
    ```

  - 只有k1无值的时候才会生效

- XX -> exist 只能更新,无法赋值

  - ```shell
    127.0.0.1:6379> set k2 redis xx
    (nil)
    127.0.0.1:6379> get k2
    (nil)
    ```

#### mset/mget 

> #### 批量设置，读取

```shell
127.0.0.1:6379> mset k3 a k4 b
OK
127.0.0.1:6379> mget k3 k4
1) "a"
2) "b"
127.0.0.1:6379>
```

#### append 

> #### 追加字符串的值

```shell
127.0.0.1:6379> get k1
"hello"
127.0.0.1:6379> append k1 " world"
(integer) 11
127.0.0.1:6379> get k1
"hello world"
```

#### getrange 

> #### 获取范围内的字符串

```shell
127.0.0.1:6379> get k1
"hello world"
127.0.0.1:6379> getrange k1 6 10
"world"
```

- 位置从`0`开始

- **反方向**索引

  - ```shell
    127.0.0.1:6379> getrange k1 -5 -1
    "world"
    ```

  - <img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210423001040.png" style="zoom:50%;" />

#### setrange 

> #### 从指定位置开始set string值

```shell
127.0.0.1:6379> setrange k1 6 SeaSoonKeun
(integer) 17
127.0.0.1:6379> get k1
"hello SeaSoonKeun"
```

#### strlen 

> #### 返回字符串长度

```shell
127.0.0.1:6379> setrange k1 6 SeaSoonKeun
(integer) 17
127.0.0.1:6379> get k1
"hello SeaSoonKeun"
127.0.0.1:6379> strlen k1
(integer) 17
```

#### type 

```bash
127.0.0.1:6379> type k1
string
127.0.0.1:6379> get k1
"hello SeaSoonKeun"
```

- Key  中的 type 包含value 类型

- Set 命令是 string 分组的，所以产生的数据都是 string类型的

  ```
  127.0.0.1:6379> set k4 999
  OK
  127.0.0.1:6379> get k4
  "999"
  127.0.0.1:6379> type k4
  string
  ```

  ```bash
  127.0.0.1:6379> help set
  
    SET key value [EX seconds|PX milliseconds] [NX|XX] [KEEPTTL]
    summary: Set the string value of a key
    since: 1.0.0
    group: string
  ```

  


