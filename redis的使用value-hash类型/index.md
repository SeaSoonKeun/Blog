# Redis-Value-Hash类型

# Redis-Value-Hash类型

## 1. Why（设计的必要性）

场景：

```bash
127.0.0.1:6379> set sean:name xcg
OK
127.0.0.1:6379> set sean:age 18
OK
127.0.0.1:6379> keys sean*
1) "sean:name"
2) "sean:age"
127.0.0.1:6379> mget sean:name
1) "xcg"
127.0.0.1:6379> mget sean:age
1) "18"
```

1. 避免keys模式匹配查询，成本比较高
2. 减少mget对多个key的取值，造成两次对服务器的通信成本。

## 2. What/HOW命令列表

与String 类型很多操作名称类似，可以进行参考

- [HDEL](https://www.redis.com.cn/commands/hdel.html)

- [HEXISTS](https://www.redis.com.cn/commands/hexists.html)

- [HGET](https://www.redis.com.cn/commands/hget.html)

- **[HGETALL](https://www.redis.com.cn/commands/hgetall.html)**

- [HINCRBY](https://www.redis.com.cn/commands/hincrby.html)

- [HINCRBYFLOAT](https://www.redis.com.cn/commands/hincrbyfloat.html)

- **[HKEYS](https://www.redis.com.cn/commands/hkeys.html)**

- [HLEN](https://www.redis.com.cn/commands/hlen.html)

- **[HMGET](https://www.redis.com.cn/commands/hmget.html)**

- **[HMSET](https://www.redis.com.cn/commands/hmset.html)**

- [HSET](https://www.redis.com.cn/commands/hset.html)

- [HSETNX](https://www.redis.com.cn/commands/hsetnx.html)

  - Redis HSETNX 命令用于为哈希表中不存在的字段赋值 。

    如果字段已经存在于哈希表中，操作无效。

    如果 key 不存在，一个新哈希表被创建并执行 HSETNX 命令。

- **[HVALS](https://www.redis.com.cn/commands/hvals.html)**

- [HSCAN](https://www.redis.com.cn/commands/hscan.html)

  - ```bash
    redis> HMSET sites google "google.com" redis "redis.com.cn" weibo "weibo.com" 4 "taobao.com"
    "OK"
    redis> HSCAN sites 0 match "red*"
    1) "0"
    2) 1) "redis"
       2) "redis.com.cn"
    ```

## 3. Where（应用场景）

​	对field进行数值计算

​	对一个key快速展开其多个value值，并快速进行计算

- 场景：点赞，收藏，详情页

eg：

```bash
127.0.0.1:6379> HMSET sean name xcg age 18 address nj
OK
127.0.0.1:6379> HMGET sean name age address
1) "xcg"
2) "18"
3) "nj"
127.0.0.1:6379> HGETALL sean
1) "name"
2) "xcg"
3) "age"
4) "18"
5) "address"
6) "nj"

```




