# 

# Redis-sharding分片

1. redis的主从复制解决了单实例的单点故障，压力问题，但是没有解决**容量的问题**
2. redis的主从复制解决了AKF的X轴问题，水平复制

先按业务逻辑分类

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/%E9%80%BB%E8%BE%91%E5%88%86%E5%8C%BA.jpg)

无法按业务分类情况下，使用sharding分片

![image-20210505232328675](/Users/mr.x/Library/Application Support/typora-user-images/image-20210505232328675.png)

- modula：hash + 取模（**弊端**，取模的数必须固定。添加节点会导致异常，扩展性较差）

- random （lpush + rpop 适用于消费队列消息订阅场景）

- Ketama一致性哈希（更加适用于缓存，而非数据库）

  > 一致性哈希[算法](https://www.xuebuyuan.com/category/算法)(Consistent Hashing Algorithm)是一种分布式算法，常用于负载均衡。Memcached client也选择这种算法，解决将key-value**均匀分配**（解决了数据倾斜问题）到众多Memcached server上的问题。它可以取代传统的取模操作，解决了取模操作无法应对增删Memcached Server的问题(增删server会导致同一个key,在get操作时分配不到数据真正存储的server，命中率会急剧下降)，
  >
  > 
  >
  > 一致性哈希算法的基本实现原理是将机器节点和key值都按照一样的hash算法映射到一个`0~2^32`的圆环上。
  > 当有一个写入缓存的请求到来时，计算 Key 值 k 对应的哈希值 Hash(k)，如果该值正好对应之前某个机器节点的 Hash 值，则直接写入该机器节点，
  > 如果没有对应的机器节点，则顺时针查找下一个节点，进行写入，如果超过 `2^32` 还没找到对应节点，则从0开始查找(因为是环状结构)。

  

  <img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210501013043.png" style="zoom:50%;" />

## Sharding机制

即通常所说的“分片”，允许数据存放在不同的物理机器上，以适应数据量过大的场景，克服单台机器内存或者磁盘空间的限制。而这种“离散式”地存放，对客户端来说是透明的，对客户端来讲，完全看不到这种差别。

## Redis的分片（Sharding或者Partitioning）技术

是指将数据分散到多个Redis实例中的方法，分片之后，每个redis拥有一部分原数据集的子集。在数据量非常大时，这种技术能够将数据量分散到若干主机的redis实例上，进而减轻单台redis实例的压力。分片技术能够以更易扩展的方式使用多台计算机的存储能力（这里主要指内存的存储能力）和计算能力：    

- （1）从存储能力的角度，分片技术通过使用多台计算机的内存来承担更大量的数据，如果没有分片技术，那么redis的存储能力将受限于单台主机的内存大小。
- （2） 从计算能力的角度，分片技术通过将计算任务分散到多核或者多台主机中，能够充分利用多核、多台主机的计算能力。





- 知识的根源
- 知识的根源

https://github.com/twitter/twemproxy


