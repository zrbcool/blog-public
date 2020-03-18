### 简述
我们知道Redis是一款优秀的内存数据库，大多数场景下，我们使用Redis作为分布式内存缓存使用，但也有部分高并发高性能场景，直接选用Redis存储数据，这就对Redis的持久化及高可用提出了要求。




### Redis同步流程
每个Redis master拥有一个replication ID，并且每个master记录一个offset并且在每个byte数据流发给slave时保持递增。

当slave连接到master时，slave利用PSYNC命令将其旧master的replication ID和当前处理到的offset发给master，如果master校验没有问题，master将只发送增量的部分。但是，如果master的backlog队列中没有足够的缓存，或者slave在请求的是一个未知的历史的offset，这时会发生一个全量同步。

所谓的全量同步，其实就是指的master节点上发生bgsave并产生一个RDB文件，与此同时申请一个buffer保存所有的新命令，当RDB文件生成完毕后，master会开始发送这个文件给slave，slave将其保存在本地磁盘并load这个文件到内存。然后master开始发送所有buffer里面保存的命令给slave执行。

这里顺便提一下SYNC命令，它与PSYNC的不同就是它不是部分的，它每次执行都是一个全量的同步，在后续版本已经不使用了，但是为了兼容性，保留了命令行的调用方式。


#### 聊聊Replication ID
实际上每个节点拥有两个replication IDs： main ID和secondary ID。
replication ID用来标记当前数据集的版本，每次一个实例作为master启动或者slave晋升作为一个master，会生成一个新的replication ID。slave在连接master之后，会继承master节点的replication ID，所以两个实例如果拥有相同的replication ID，并且offset相同，则说明他们当前的数据相同。

那么为什么每个节点有两个replication IDs呢，因为当发生failover，某个slave晋升为master，此时其他的slaves会以旧的replication ID来请求同步，这时晋升的slave，也就是新master是知道旧master的replication ID的，它用secondary ID作为main ID，也就是继承了旧master的replication ID，得以保障主从复制可以继续。
#### 聊聊复制积压缓冲区 - backlog
先看一下配置
```
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
normal 0 0 0 
slave 256mb 64mb 60 
pubsub 32mb 8mb 60
```
意思是如果slave缓冲区的大小超过256mb，或该缓冲区的大小超过64mb并且持续60s，会立刻断开slave的连接。slave在repl-timeout时间后发现没有同步任何数据，重新发起PSYNC同步请求。

相关配置如下：
```
# The following option sets the replication timeout for:
#
# 1) Bulk transfer I/O during SYNC, from the point of view of slave.
# 2) Master timeout from the point of view of slaves (data, pings).
# 3) Slave timeout from the point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-slave-period otherwise a timeout will be detected
# every time there is low traffic between the master and the slave.
repl-timeout 60
# Set the replication backlog size. The backlog is a buffer that accumulates
# slave data when slaves are disconnected for some time, so that when a slave
# wants to reconnect again, often a full resync is not needed, but a partial
# resync is enough, just passing the portion of data the slave missed while
# disconnected.
#
# The bigger the replication backlog, the longer the time the slave can be
# disconnected and later be able to perform a partial resynchronization.
#
# The backlog is only allocated once there is at least a slave connected.
repl-backlog-size 1mb
repl-backlog-ttl 3600
```
# 查看当前Replication状态
```
role:master
connected_slaves:1
slave0:ip=172.16.13.128,port=9720,state=online,offset=7885515926852,lag=1
master_replid:e44c50848619ddcf19dc140466519e8309727c81
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:7885516112507
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:7885515063932
repl_backlog_histlen:1048576
```
### 生产实践建议

1）主节点的bgsave操作，会给主节点带来一个短时间的性能抖动，原因Redis的主进程调用fork函数时是一个同步操作，主进程会等待fork函数返回才继续执行后面的代码，当内存较大时，fork的返回时间会变长。
优化：由于fork函数的执行时间与内存使用量及机型有关，所以控制单个节点的内存大小就很关键，我们的事件是控制单个节点的maxmemory为8GB，这样可以有效控制bgsave的fork调用产生的性能抖动。
2）写RDB文件时，会产生磁盘IO，也会带来性能的消耗。
优化：对于从节点这个性能消耗可以忽略，所以可以关闭主节点的bgsave，并在从节点开启。但是前面提到的全量同步时发生bgsave生成的RDB文件，在某些磁盘压力大的系统下会导致糟糕的性能表现，所以在2.8.18版本以后，开始支持不需要落盘的全量同步模式。具体参考配置项：repl-diskless-sync及repl-diskless-sync-delay

### 相关技术简介
#### bgsave
bgsave原理是使用操作系统的fork函数，产生一个子进程，子进程拥有当前主进程的内存快照（实际上不是复制了全部的内存内容而是基于COW），父进程通过COW（Copy On Write）技术操作内存，在修改前将修改的page页进行拷贝，再行修改，这样就不会影响子进程看到的内存内容，子进程就可以异步的将内存中的数据序列化并写入磁盘。 
#### COW - Copy On Write
在复制一个对象的时候并不是真正的把原先的对象复制到内存的另外一个位置上，而是在新对象的内存映射表中设置一个指针，指向源对象的位置，并把那块内存的Copy-On-Write位设置为1.

这样，在对新的对象执行读操作的时候，内存数据不发生任何变动，直接执行读操作；而在对新的对象执行写操作时，将真正的对象复制到新的内存地址中，并修改新对象的内存映射表指向这个新的位置，并在新的内存位置上执行写操作。

这个技术需要跟虚拟内存和分页同时使用，好处就是在执行复制操作时因为不是真正的内存复制，而只是建立了一个指针，因而大大提高效率。但这不是一直成立的，如果在复制新对象之后，大部分对象都还需要继续进行写操作会产生大量的分页错误，得不偿失。所以COW高效的情况只是在复制新对象之后，在一小部分的内存分页上进行写操作。

传统的fork函数直接把所有资源复制给新的进程，效率很低下。
写时拷贝在需要写入时，数据才会被复制，没有数据写入时，fork()的开销实际只是复制父进程的页表以及给子进程创建唯一的进程描述符。有数据要写入前，会将将要改变的数据页复制给子进程。

详细参考：http://www.cnblogs.com/biyeymyhjob/archive/2012/07/20/2601655.html
### 相关参考
https://redis.io/topics/persistence
https://redis.io/topics/replication
https://www.cnblogs.com/lukexwang/p/4711977.html
http://www.cnblogs.com/biyeymyhjob/archive/2012/07/20/2601655.html
### 作者其他文章
[https://github.com/zrbcool/blog-public](https://github.com/zrbcool/blog-public)  
### 微信订阅号
![](http://oss.zrbcool.top/Fv816XFbZB2JQazo5LHBoy2_SGVz)