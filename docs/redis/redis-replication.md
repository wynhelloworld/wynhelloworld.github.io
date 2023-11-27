# Redis 主从复制

Redis 主从复制，是指将一台 Redis 服务器上的数据，复制到其它 Redis 服务器上。前者称为主节点（Master），后者称为从节点（Slave）。

主节点上的数据会同步给从节点，而从节点不能同步给主节点。也就是说主节点能够进行读写，而从节点只能进行读（当然通过配置文件也可以修改成可写）。所以，Redis 主从复制的作用是体现在了读操作上，而不是写操作上。

## 主从复制的作用

当只有一台 Redis 服务器时，会容易出现以下问题：

- 当 Redis 服务器达到性能瓶颈时，会暂时无法处理新的命令请求。
- 当 Redis 服务器宕机时，由于数据恢复需要时间，在此期间，也无法处理新的命令请求。
- 当 Redis 服务器硬盘出现故障时，会导致数据丢失。

主从复制就解决了上述问题。主节点将数据同步到从节点上，配合负载均衡，能大大降低主节点的压力。即使其中的某个节点挂掉，也不会中断整个服务。即使其中的某个节点硬盘出现故障，也不会导致整个服务的数据丢失。

## 在单服务器上构建 Redis 主从复制

这里以创建一个从节点为例（创建多个从节点，重复该步骤即可）：

创建从节点配置文件

```shell
sudo cp /etc/redis/redis.conf /etc/redis/slave6380.conf
```

加上以下内容：

```shell
slaveof 127.0.0.1 6379
```

修改以下内容：

```shell
port 6380
daemonize yes
dir /var/lib/redis/slave6380
```

创建从节点工作目录

```shell
sudo mkdir /var/lib/redis/slave6380
```

启动从节点

```shell
redis-cli /etc/redis/slave6380.conf
```

## 查看主从结构信息

通过 `info replication` 命令可查看主从结构信息，信息如下（左边是主节点，右边是从节点）：

![image-20231127210201124](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127210201124.png)

- slave0：描述了第一个从节点的信息
  - offset：描述了该节点的更新进度
- master_replid：标识一个主节点，当主节点启动或者是从节点晋升为主节点时生成
- master_replid2：当从节点与主节点异常断开时，从节点会晋升为主节点，然后利用 replid2 保存下主节点的 replid（为全 0 时，表示该选项还未作用），当从节点重新与主节点恢复连接时要用曾经用 replid2 保存下的 replid（哨兵机制）
- master_repl_offset：描述了主节点的数据的总进度
- slave_repl_offset：描述了从节点的数据的更新进度

replid 和 offset 共同标识了一块数据集，若是两个节点的 replid 和 offset 都相等，则说明这两个节点的数据完全一样。

## 更改主从结构

通过 `slaveof`命令可更改主从结构。

- 从节点执行 `slaveof no one`，表示永久脱离主节点，该节点成为一个独立的节点；
- 从节点执行 `slaveof ip port`，表示要成为该 redis-server 的从节点。

但是要注意，在命令行中敲这些命令只是暂时的，当 redis-server 重启时，会重新读配置文件，原本是谁的从节点，重启后还是谁的从节点。

## 拓扑结构

**一主一从**

![image-20231127212409652](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127212409652.png)

此结构是最简单的复制拓扑结构，用于当主节点发生故障宕机时，从节点提供故障转移支持。当主节点的“写命令”并发量较高且需要持久化时，可以选择主节点不开启而从节点开启 AOF。但此时主节点重启时需要先获取从节点的 AOF 日志。

**一主多从**

![image-20231127212422170](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127212422170.png)

此结构实现了“读写分离”，适用于读命令并发量高的场景，但是当写命令并发量高时，由于主节点要重复的写很多份的写命令，所以会严重影响主节点的带宽。

**树状主从**

![image-20231127212506061](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127212506061.png)

此结构是一主多从结构的优化版，当写命令并发量高时，主节点的带宽并不会很高，由中间层的从节点向底层的从节点同步命令。

## 主从复制的建立流程

![image-20231127214954887](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127214954887.png)



## psync 同步数据

Redis 使用 psync 命令完成主从节点的数据同步，数据同步分为两部分：全量复制、部分复制。

全量复制：一般用于主从节点初次建立连接时，此时主节点会生成 RDB 并发送给从节点。

部分复制：一般用于当主从节点因意外断开后又重新连接时，由于此时从节点已有大部分主节点的数据，所以不需要再次将主节点的全部数据同步给从节点，只需要同步新的那部分数据。

#### psync 的语法格式

```c++
psync replid offset
```

当 replid 为 ? ，offset 为 -1 时，就是在尝试全量复制。

当 replid 为具体值，offset >= 0 时，就是在尝试部分复制。

#### psync 的运行流程

![image-20231127220250668](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127220250668.png)

1、首先，从节点发送 psync 请求，默认值为 ？ -1；

2、主节点发送响应，若为 fullresync，则全量复制；

​				     若为 continue，则部分复制；

​				     若为 err，则主节点版本过低，不支持 psync 命令，从节点可尝试用 sync 命令。

> psync 会在主从复制建立的过程中自动被调用；
>
> psync 不会阻塞主线程，sync 会阻塞主线程。



<script src="https://giscus.app/client.js"
        data-repo="wynhelloworld/blog-comments"
        data-repo-id="R_kgDOKruZpg"
        data-category="Announcements"
        data-category-id="DIC_kwDOKruZps4Ca2L0"
        data-mapping="url"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>

本站所有文章转发 **CSDN** 将按侵权追究法律责任，其它情况随意。
