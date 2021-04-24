# Redis介绍和安装


<!--more-->

## 1. 常识
- 磁盘：
    - 寻址：ms
    - 带宽：G/M
- 内存：ns
    - 寻址：ns 10万倍 ms us ns
    - 带宽：G/M    
- IO buffer成本问题
    - 磁盘 磁道 扇区512Bytes 造成索引量很大
    - 操作系统默认以4K为最小单位进行读取

## 2. 数据存储发展进程
grep awk java读取文件里的数据时，会随着数据量的变大，速度变慢。因为硬盘IO成为瓶颈；
### 数据库引擎
https://db-engines.com/en/


### 编译：源码变为可执行程序

## 3. 安装
```shell
1,yum install wget
2,cd ~
3,mkdir soft
4,cd soft
5,wget    http://download.redis.io/releases/redis-5.0.5.tar.gz
6,tar xf    redis...tar.gz
7,cd redis-src
8,看README.md
9, make 
 ....yum install  gcc  
....  make distclean
10,make
11,cd src   ....生成了可执行程序
12, cd ..
13,make install PREFIX=/opt/mashibing/redis5
14,vi /etc/profile
...   export  REDIS_HOME=/opt/mashibing/redis5
...   export PATH=$PATH:$REDIS_HOME/bin
..source /etc/profile
15,cd utils
16,./install_server.sh  （可以执行一次或多次）
    a)  一个物理机中可以有多个redis实例（进程），通过port区分
    b)  可执行程序就一份在目录，但是内存中未来的多个实例需要各自的配置文件，持久化目录等资源
    c)  service   redis_6379  start/stop/stauts     >   linux   /etc/init.d/**** 
    d)脚本还会帮你启动！
17,ps -fe |  grep redis  



```
### centos7编译报错
```shell
server.c:5307:31: 错误:‘struct redisServer’没有名为‘server_xxx’的成员
 

redis编译报上面错误，一般是环境问题，执行下面操作可以解决

1、安装gcc套装：

yum install cpp
yum install binutils
yum install glibc
yum install glibc-kernheaders
yum install glibc-common
yum install glibc-devel
yum install gcc
yum install make

2、升级gcc

yum -y install centos-release-scl

yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

scl enable devtoolset-9 bash

3、设置永久升级：

echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile

4、安装redis：

```
### centos7 执行utils/install_server.sh 因为systemctl命令报错，注释掉即可

![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/centos7%E5%AE%89%E8%A3%85redis%E8%84%9A%E6%9C%AC%E6%8A%A5%E9%94%991.jpg)
