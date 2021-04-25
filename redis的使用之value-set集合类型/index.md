# Redis-Value-Set集合类型

# Redis-Value-Set集合类型

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/set1.jpg)

## 1. Why

### 特点

- 无序
- 去重

### 应用场景

- 集合操作。求交、并集和差集

- 随机事件。

  - 解决**抽奖问题**：10个奖品 用户<10，用户中奖分为是否重复

    - **正数去重**：符合一个人只能中一个礼物的语义

    - **负数可以重复**： 

    - 几个**维度**：

      ![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/srandmember.jpg)

- 可以使用Redis的Set数据类型**跟踪一些唯一性数据**，比如访问某一博客的唯一IP地址信息。对于此场景，我们仅需在每次访问该博客时将访问者的IP存入Redis中，Set数据类型会自动保证IP地址的唯一性。

- 充分利用Set类型的服务端聚合操作方便、高效的特性，可以用于**维护数据对象之间的关联关系**。比如所有购买某一电子设备的客户ID被存储在一个指定的Set中，而购买另外一种电子产品的客户ID被存储在另外一个Set中，如果此时我们想获取有哪些客户同时购买了这两种商品时，Set的**intersections交集命令**就可以充分发挥它的方便和效率的优势了。

## 2. What

### 1. help @set

```bash
SADD key member [member ...]
  summary: Add one or more members to a set
  since: 1.0.0

  SCARD key
  summary: Get the number of members in a set
  since: 1.0.0

  SDIFF key [key ...]
  summary: Subtract multiple sets
  since: 1.0.0

  SDIFFSTORE destination key [key ...]
  summary: Subtract multiple sets and store the resulting set in a key
  since: 1.0.0

  SINTER key [key ...]
  summary: Intersect multiple sets
  since: 1.0.0

  SINTERSTORE destination key [key ...]
  summary: Intersect multiple sets and store the resulting set in a key
  since: 1.0.0

  SISMEMBER key member
  summary: Determine if a given value is a member of a set
  since: 1.0.0

  SMEMBERS key
  summary: Get all the members in a set
  since: 1.0.0

  SMOVE source destination member
  summary: Move a member from one set to another
  since: 1.0.0

  SPOP key [count]
  summary: Remove and return one or multiple random members from a set
  since: 1.0.0

  SRANDMEMBER key [count]
  summary: Get one or multiple random members from a set
  since: 1.0.0

  SREM key member [member ...]
  summary: Remove one or more members from a set
  since: 1.0.0

  SSCAN key cursor [MATCH pattern] [COUNT count]
  summary: Incrementally iterate Set elements
  since: 2.8.0

  SUNION key [key ...]
  summary: Add multiple sets
  since: 1.0.0

  SUNIONSTORE destination key [key ...]
  summary: Add multiple sets and store the resulting set in a key
  since: 1.0.0
```

### 2. 文档扩展

- [SADD](https://www.redis.com.cn/commands/sadd.html)

- [SCARD](https://www.redis.com.cn/commands/scard.html)

- [SDIFF](https://www.redis.com.cn/commands/sdiff.html) 按**传递的参数顺序**进行取**左差**集

- [SDIFFSTORE](https://www.redis.com.cn/commands/sdiffstore.html) STORE方法是，作者设计的细腻之处，**避免多次IO的产生**

- [SINTER](https://www.redis.com.cn/commands/sinter.html)

- [SINTERSTORE](https://www.redis.com.cn/commands/sinterstore.html)

- [SISMEMBER](https://www.redis.com.cn/commands/sismember.html) 

- [SMEMBERS](https://www.redis.com.cn/commands/smembers.html) 较消耗资源的操作

- [SMOVE](https://www.redis.com.cn/commands/smove.html)

- **[SPOP](https://www.redis.com.cn/commands/spop.html)** 取出一个

- **[SRANDMEMBER](https://www.redis.com.cn/commands/srandmember.html)**

  - Redis [SRANDMEMBER](https://www.redis.com.cn/commands/srandmember.html) 命令仅使用`key` 参数，那么随机返回集合`key` 中的一个随机元素。

    Redis 2.6开始，可以接受 count 参数，如果count是整数且**小于**元素的个数，返回含有 count 个**不同的**元素的数组，如果count是个整数且大于集合中元素的个数时，返回整个集合的所有元素，当count是**负数**，则会返回一个包含count的绝对值的个数元素的数组，则返回的结果集里会出现一个元素出现多次的情况。

    仅提供key参数时，该命令作用类似于`SPOP`命令，不同的是`SPOP`命令会将被选择的随机元素从集合中移除，而`SRANDMEMBER` 仅仅是返回该随记元素，而不对原集合做任何操作。

    #### 1.  传递count参数时的行为规范

    当传递了一个值为正数的count参数，返回的元素就好像从集合中移除了每个选中的元素一样（就像在宾果游戏中提取数字一样）。但是**元素不会从集合中移除**。所以基本上：

    - 不会返回重复的元素。
    - 如果count参数的值大于集合内的元素数量，此命令将会仅返回整个集合，没有额外的元素。

    相反，当count参数的值**为负数时，此命令的行为将发生改变，并且提取操作就像在每次提取后，重新将取出的元素放回包里一样，因此，可能返回重复的元素**，以及总是会返回我们请求的数量的元素，因为我们可以一次又一次地重复相同的元素，除了当集合为空（或者不存在key）的时候，将总是会返回一个空数组。

    

    #### 2. 返回元素的分布

    当集合中的元素数量很少时，返回元素分布远不够完美，这是因为我们使用了一个近似随机元素函数，它并不能保证良好的分布。

    所使用的算法（在dict.c中实现）对**哈希表桶进行采样以找到非空桶**。一旦找到非空桶，由于我们在哈希表的实现中使用了链接法，因此会检查桶中的元素数量，并且选出一个随机元素。

    这意味着，如果你在整个哈希表中有两个非空桶，其中一个有三个元素，另一个只有一个元素，那么其桶中单独存在的元素将以更高的概率返回。

- [SREM](https://www.redis.com.cn/commands/srem.html)

- [SUNION](https://www.redis.com.cn/commands/sunion.html)

- [SUNIONSTORE](https://www.redis.com.cn/commands/sunionstore.html)

- [SSCAN](https://www.redis.com.cn/commands/sscan.html)

### 3. 分类

 - 增删改查

 - 集合操作 交集，并集，差集

 - 随机事件

   


