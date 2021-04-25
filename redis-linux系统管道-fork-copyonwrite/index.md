# Redis-fork-cow-引申linux系统管道

# linux系统管道

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/linux%20%E7%B3%BB%E7%BB%9F%E7%AE%A1%E9%81%93%E5%BC%95%E7%94%B3%E7%88%B6%E5%AD%90%E8%BF%9B%E7%A8%8B%E7%9A%84fork()%E5%85%B3%E7%B3%BB.jpg)

## 1. 管道符

1，衔接，前一个命令的输出作为后一个命令的输入

2，管道会触发创建【子进程】

## 2. 环境变量，父子进程的变量空间

进阶思想，父进程其实可以让子进程看到数据！

linux中

export的环境变量，子进程的修改不会破坏父进程

父进程的修改也不会破坏子进程

## 3. fork()

1，速度：快

2，空间：小

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/fork.jpg)

## 4. copy on write 写时复制 - 加快创建子进程速度

**创建**子进程并**不发生复制**，只有在想**修改数据的时候**才会去定向复制一部分数据。

优势：**创建进程变快**，同时根据经验，子进程不可能把父进程所有数据都改一遍。

主要是针对**`指针`**的操作
