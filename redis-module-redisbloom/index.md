# Redis-Module-Bloom-白名单机制


# Redis-Module-Bloom-**白名单**机制

位图的最常见场景之一: 大数据下去重过滤功能.

## 1. Why

你有啥？有的数据都拿到一个集合里，把用户搜索的和你已有的做一个比对，如果没找到就不用去数据库了。

 但是像淘宝这种网站，本身数据库的数据就已经需要分而治之了，内存数据库更难以实现了，所以诞生了布隆控制器。



（有点类似于**白名单**）

解决**缓存穿透**问题



## 2. What

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210425171702.png)

### 原理：

布隆过滤器原理是通过多个 hash 函数, 将结果对应的位设为1, 所以它能做到 100%的去重. 虽然会误判, 但是在合理的设计的前提下, 误判率是可以接受的.

实现代码：

<img src="/Users/mr.x/Library/Application Support/typora-user-images/image-20210425192612228.png" alt="image-20210425192612228" style="zoom:50%;" />



**牺牲数据的精度，换取空间效率**。会出现不同的值算出来的值是一样的情况。

### 缺点：

1. 首先它的**查询性能相对较弱**, 它是用 hash 函数在位图上不同的点跳跃探测, **很难利用 cpu 缓存**.
2. 第二它**不支持删除**, 假如邮箱a 的 hash 结果分别是 1 3 7, 邮箱 b 的结果是 2 6 7. 那么在删除邮箱a 的结果之后, 邮箱 b 也会被重新认为是不存在, 因为第 7 位的值是 0. 这一点布隆过滤器也很无奈.当位图拥挤到一定程度, 只能选择**重建整个位图**.
3. 存在**误判**概率

## 3. How

### 	1. 下载，编译，并加载布隆过滤器

#### 

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8.jpg" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/bloom1.jpg" style="zoom:30%;" />



```bash
# 下载bloom模块并编译
wget https://github.com/RedisBloom/RedisBloom/archive/refs/heads/master.zip
unzip master.zip
make生成redisbloom.so扩展库


# 启动redis服务
service redis_6379 stop
[root@localhost bin]# ./redis-server /etc/redis/6379.conf --loadmodule /opt/soft/redis/redisbloom.so
[root@localhost bin]# ps -ef |grep redis
root     29460     1  0 00:10 ?        00:00:00 ./redis-server 127.0.0.1:6379

# 连接
[root@localhost bin]# redis-cli
127.0.0.1:6379> BF.LOADCHUNK key ...options...
可显示bf扩展关键字
```

注意加载模块时配置**绝对路径**

### 2. 布隆过滤器的思路：小空间解决大量数据匹配问题

1. 

   ![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/bloom%E6%A0%B8%E5%BF%83.jpg)

   bitmap二进制位进行代表，会使得**体积变得很小**

   `**所有映射函数的值都是1则进行穿透，有一个映射函数遇到0就不往下查询了`**

   **运算的复杂度 + 空间 换取 查询成本**

2. **概率解决问题**，不可能百分百阻挡，未阻挡<1%。函数的数量和bitmap数组的宽度需要调整。

## 4. 补充知识点

1. 查询穿透缓存了，但是查询数据库不存在值，client，增加redis中的key，value标记

2. 数据库增加了新元素，需完成元素对bloom的添加 涉及到**双写问题**

## 5. 发展

- bloom -> counting bloom -> cuckoo **布谷鸟过滤器**

  bloom过滤器问题：

1. 首先它的**查询性能相对较弱**, 它是用 hash 函数在位图上不同的点跳跃探测, **很难利用 cpu 缓存**.
2. 第二它**不支持删除**, 假如邮箱a 的 hash 结果分别是 1 3 7, 邮箱 b 的结果是 2 6 7. 那么在删除邮箱a 的结果之后, 邮箱 b 也会被重新认为是不存在, 因为第 7 位的值是 0. 这一点布隆过滤器也很无奈.当位图拥挤到一定程度, 只能选择**重建整个位图**.

### counting bloom：

原理：把**位图**的位 升级为**计数器(Counter)**. 添加元素, 就给对应的Counter分别+1; 删除元素, 就给对应的Counter分别减一. 用多出几倍存储空间的代价, 来实现删除功能. 虽然实现了功能, 但是不够完美.

### Cuckoo filter:

[论文地址](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf)

> 首先要说明布谷鸟过滤器并不是使用位图实现的, 而是**一维数组.** 它所存储的是数据的指纹(fingerprint).
> 布谷鸟过滤器使用两个 hash 算法将新来的元素映射到数组的两个位置. 如果两个位置中有一个位置位空, 那么就可以将元素直接放进去. 但是如果这两个位置都满了, 它就会随机踢走一个, 然后自己霸占了这个位置.
> 正如布谷鸟那样, 把蛋下到其它鸟的窝里. 这也是得名的由来. 但它并不是像布谷鸟那样, 管杀不管埋, 还会为这个被踢走的数据, 找一个新家.
> 这里看一下它的公式, Cuckoo filter中只采用两个哈希映射函数 H1 和 H2，H3用于计算数据的 fingerprint. 他们的关系如下
>
> H3(key) = key’s fingerprint = hash(key)
> H1(key) = hash1(key)
> H2(key) = H1(key) ^ H1(key’s fingerprint) // 异或
>
> 从上面的公式中可以看出，当我们知道 fingerprint 和 H1(key)，就可以直接算出 H2(key)。同样如果我们知道 H2(key) 和 fpfingerprint 也可以直接算出H1(key) ---- 对偶性.
> 通过 fingerprint 和当前位置, 算出对应的另一个巢, 然后安放这个可怜的数据. 如果另一个巢仍然有数据, 那就为受害者继续寻找下一个巢.
> 但也有问题, 假如数组太过拥挤, 踢了几十次仍然没有找到空缺的巢, 那就需要为数组扩容了.
> 不过原生的布谷鸟过滤器空间利用率并不高, 大约 50%. 改良的方案之一是增加 hash 函数, 让每个元素不止有两个巢, 这样可以大大降低碰撞的概率, 将空间利用率提高到 95% 左右.
> 还有个方案是给每个位置上挂多个巢, 这样不会马上就挤来挤去. 也能大大降低碰撞概率, 空间利用率虽然比第一种改良方案稍低(约为 85%), 但cpu 缓存的利用率会提高不少.
> 甚至还可以将两种方案结合, 据说空间利用率高达99%. 了不起的数字.
> 要删除也很简单, 找到对应位置的指纹信息删除即可.
> 但布谷鸟过滤器有一个明显的弱点, 无法对同一个数据连续插入!
> 刚才也说到了, 假如数组太拥挤, 碰撞多次仍然未找到空缺的巢, 那就需要扩容了. 那么连续插入同一个数据, 马上就触发了扩容.
> 而且因为存储的只是一个字节的指纹信息, 也很难判断插入的数据和位置上的数据, 是不是同一个数据. 算法论文上也有说明, 支持删除, 不支持同一个数据多次插入. 要确保每一个元素不会被插入多次(kb+1). k 是指 hash 函数的个数 2，b是指单个位置上的座位数. 也就是说原生布谷鸟过滤器, 不能超过 3.

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/%E5%B8%83%E8%B0%B7%E9%B8%9Fhash.jpg)

- 布谷鸟问题：

  1. **循环挤兑**问题；插入重复数据不能超过 `hash函数个数` * `每个数组包含的子数组个数` + `1`次，想实现这点还需要另外一层数据结构来进行**维护次数**，增加了开销。而且如果一个指纹使用一个字节，只有256种可能性，会出现不同数据相同指纹的情况，也会出现一定概率的**误判情况**。

  2. 鸠占鹊巢的次数有限制，到一定次数后，会进行**数组扩容**；类似于hashmap，达到0.75的时候进行扩容

  3. 删除功能缺陷：一个指纹使用一个字节，只有256种可能性，也有不同数据相同指纹的情况，会有**误删除**其他数据的情况发生。



- 优化方法：

  1. **hash函数个数**，可以有多个位置可以选择

  2. **1嵌套内部子数组**

  3. 以通过**调整指纹信息的保存量**来降低误判情况，如在上面的实现中，指纹信息是 1byte 保存8位信息误判概率是0.03，当指纹信息增加到 2bytes 保存16位信息误判概率会降低至 0.0001。



## 6. 缓存雪崩：

​	方法：缓存不设置相同的超时时间，使用定时任务定期进行更新。

## 7. 黑名单

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210425171702.png)





## 8. 使用场景

### 黑名单：

- 视频网站推送视频
- <img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210425173746.png" style="zoom:50%;" />

### 白名单：

- 允许转载
- <img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210425174042.png" style="zoom:50%;" />




