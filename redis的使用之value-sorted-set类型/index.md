# redis-Value-sorted-set类型

# redis-Value-sorted-set类型

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/sorted_set.jpg" style="zoom:50%;" />

## 1. Why

### 特点

- 有序
- 分数 scores 进行排序

## 2. What

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/sorted_set%E5%90%AF%E8%92%99.jpg" style="zoom:50%;" />

### 1. Help @sorted_set

```bash
127.0.0.1:6379> help @sorted_set

  BZPOPMAX key [key ...] timeout
  summary: Remove and return the member with the highest score from one or more sorted sets, or block until one is available
  since: 5.0.0

  BZPOPMIN key [key ...] timeout
  summary: Remove and return the member with the lowest score from one or more sorted sets, or block until one is available
  since: 5.0.0

  ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
  summary: Add one or more members to a sorted set, or update its score if it already exists
  since: 1.2.0

  ZCARD key
  summary: Get the number of members in a sorted set
  since: 1.2.0

  ZCOUNT key min max
  summary: Count the members in a sorted set with scores within the given values
  since: 2.0.0

  ZINCRBY key increment member
  summary: Increment the score of a member in a sorted set
  since: 1.2.0

  ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
  summary: Intersect multiple sorted sets and store the resulting sorted set in a new key
  since: 2.0.0

  ZLEXCOUNT key min max
  summary: Count the number of members in a sorted set between a given lexicographical range
  since: 2.8.9

  ZPOPMAX key [count]
  summary: Remove and return members with the highest scores in a sorted set
  since: 5.0.0

  ZPOPMIN key [count]
  summary: Remove and return members with the lowest scores in a sorted set
  since: 5.0.0

  ZRANGE key start stop [WITHSCORES]
  summary: Return a range of members in a sorted set, by index
  since: 1.2.0

  ZRANGEBYLEX key min max [LIMIT offset count]
  summary: Return a range of members in a sorted set, by lexicographical range
  since: 2.8.9

  ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
  summary: Return a range of members in a sorted set, by score
  since: 1.0.5

  ZRANK key member
  summary: Determine the index of a member in a sorted set
  since: 2.0.0

  ZREM key member [member ...]
  summary: Remove one or more members from a sorted set
  since: 1.2.0

  ZREMRANGEBYLEX key min max
  summary: Remove all members in a sorted set between the given lexicographical range
  since: 2.8.9

  ZREMRANGEBYRANK key start stop
  summary: Remove all members in a sorted set within the given indexes
  since: 2.0.0

  ZREMRANGEBYSCORE key min max
  summary: Remove all members in a sorted set within the given scores
  since: 1.2.0

  ZREVRANGE key start stop [WITHSCORES]
  summary: Return a range of members in a sorted set, by index, with scores ordered from high to low
  since: 1.2.0

  ZREVRANGEBYLEX key max min [LIMIT offset count]
  summary: Return a range of members in a sorted set, by lexicographical range, ordered from higher to lower strings.
  since: 2.8.9

  ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
  summary: Return a range of members in a sorted set, by score, with scores ordered from high to low
  since: 2.2.0

  ZREVRANK key member
  summary: Determine the index of a member in a sorted set, with scores ordered from high to low
  since: 2.0.0

  ZSCAN key cursor [MATCH pattern] [COUNT count]
  summary: Incrementally iterate sorted sets elements and associated scores
  since: 2.8.0

  ZSCORE key member
  summary: Get the score associated with the given member in a sorted set
  since: 1.2.0

  ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
  summary: Add multiple sorted sets and store the resulting sorted set in a new key
  since: 2.0.0
```



### 2. Redis 有序集合(sorted set) 命令

| 命令                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Redis Zrevrank 命令](https://www.redis.net.cn/order/3625.html) | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| [Redis Zlexcount 命令](https://www.redis.net.cn/order/3614.html) | 在有序集合中计算指定字典区间内成员数量                       |
| [Redis Zunionstore 命令](https://www.redis.net.cn/order/3627.html) | 计算给定的一个或多个有序集的并集，并存储在新的 key 中        |
| [Redis Zremrangebyrank 命令](https://www.redis.net.cn/order/3621.html) | 移除有序集合中给定的排名区间的所有成员                       |
| [Redis Zcard 命令](https://www.redis.net.cn/order/3610.html) | 获取有序集合的成员数                                         |
| [Redis Zrem 命令](https://www.redis.net.cn/order/3619.html)  | 移除有序集合中的一个或多个成员                               |
| [Redis Zinterstore 命令](https://www.redis.net.cn/order/3613.html) | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中 |
| [Redis Zrank 命令](https://www.redis.net.cn/order/3618.html) | 返回有序集合中指定成员的索引                                 |
| [Redis Zincrby 命令](https://www.redis.net.cn/order/3612.html) | 有序集合中对指定成员的分数加上增量 increment                 |
| [Redis Zrangebyscore 命令](https://www.redis.net.cn/order/3617.html) | 通过分数返回有序集合指定区间内的成员                         |
| [Redis Zrangebylex 命令](https://www.redis.net.cn/order/3616.html) | 通过字典区间返回有序集合的成员                               |
| [Redis Zscore 命令](https://www.redis.net.cn/order/3626.html) | 返回有序集中，成员的分数值                                   |
| [Redis Zremrangebyscore 命令](https://www.redis.net.cn/order/3622.html) | 移除有序集合中给定的分数区间的所有成员                       |
| [Redis Zscan 命令](https://www.redis.net.cn/order/3628.html) | 迭代有序集合中的元素（包括元素成员和元素分值）               |
| [Redis Zrevrangebyscore 命令](https://www.redis.net.cn/order/3624.html) | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
| [Redis Zremrangebylex 命令](https://www.redis.net.cn/order/3620.html) | 移除有序集合中给定的字典区间的所有成员                       |
| [Redis Zrevrange 命令](https://www.redis.net.cn/order/3623.html) | 返回有序集中指定区间内的成员，通过索引，分数从高到底         |
| [Redis Zrange 命令](https://www.redis.net.cn/order/3615.html) | 通过索引区间返回有序集合成指定区间内的成员                   |
| **[Redis Zcount 命令](https://www.redis.net.cn/order/3611.html)** | 计算在有序集合中指定区间分数的成员数                         |
| **[Redis Zadd 命令](https://www.redis.net.cn/order/3609.html)** | 向有序集合添加一个或多个成员，或者更新已存在成员的分数       |

## 3. How

在Sorted-Set中添加、删除或更新一个成员都是**非常快速**的操作，其时间复杂度为集合中成员数量的对数。由于Sorted-Set中的成员在集合中的位置是有序的，因此，即便是访问位于集合中部的成员也仍然是非常高效的。

### 1. 跳表

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/%E8%B7%B3%E8%A1%A8%E5%90%AF%E8%92%99.jpg)

## 4. Where

### 1. 排行榜

可以用于一个大型在线游戏的积分**排行榜**。每当玩家的分数发生变化时，可以执行ZADD命令更新玩家的分数，此后再通过ZRANGE命令获取积分TOPTEN的用户信息。当然我们也可以利用`ZRANK`命令通过username来获取玩家的排行信息。**最后我们将组合使用ZRANGE和ZRANK命令快速的获取和某个玩家积分相近的其他用户的信息。**

### 2. Sorted-Set类型还可用于构建索引数据。


