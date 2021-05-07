# 

# redis中代理的使用

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210501015951.png)



预分区



数据分治的弊端：难实现聚合操作。事务



predixy 代理 - 作者神秘，相较于其他代理，功能比较全面。





实操：

代理：解耦后面的复杂度，client不用管后面复杂的东西



击穿：少量的key高并发，引起数据库访问量激增（较极端）

穿透：布隆过滤器（三种应用位置）

雪崩：大量的key同时失效，间接造成大量访问到达数据库（均匀分布过期时间）



分布式锁



api：1.连接 2.高低阶命令 3. 编码serialzer序列化问题

redistemplate

stringRedisTemplate



所有类型分为两类

单值的，复值的


