# Redis 主从复制

Redis 主从复制，是指将一台 Redis 服务器上的数据，复制到其它 Redis 服务器上。前者称为主节点（Master），后者称为从节点（Slave）。

Master 上的数据会同步给 Slave，而 Slave 上的数据不能同步给 Master。也就是说 Master 能够处理读命令和写命令，而 Slave 只能处理读命令（当然通过修改配置文件也能够使得 Slave 处理写命令）。所以，Redis 主从复制的作用是体现在了处理读命令上，而不是处理写命令上。

## 主从复制的作用

当只有一台 Redis 服务器时，会容易出现以下问题：

- 当 Redis 服务器达到性能瓶颈时，会暂时无法处理新的命令。
- 当 Redis 服务器宕机时，由于数据恢复需要时间，在此期间，也无法处理新的命令。
- 当 Redis 服务器硬盘出现故障时，会导致数据丢失。

主从复制就解决了上述问题。Master 将数据同步到 Slave 上，配合负载均衡，能大大降低 Master 的压力。即使其中的某个节点挂掉，也不会中断整个服务。即使其中的某个节点硬盘出现故障，也不会导致整个服务的数据丢失。

## 在单服务器上构建 Redis 主从复制结构

这里以创建一个 Slave 为例（创建多个 Slave，重复该步骤即可）：

创建 Slave 配置文件，这里取名为 slave6380.conf：

```shell
sudo cp /etc/redis/redis.conf /etc/redis/slave6380.conf
```

添加以下内容：

```shell
slaveof 127.0.0.1 6379
```

修改以下内容：

```shell
port 6380
daemonize yes
dir /var/lib/redis/slave6380
```

创建 Slave 工作目录：

```shell
sudo mkdir /var/lib/redis/slave6380
```

启动从节点：

```shell
redis-cli /etc/redis/slave6380.conf
```

## 查看主从结构信息

通过 `info replication` 命令可查看主从结构信息，信息如下：

![image-20231127210201124](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127210201124.png)

- slave0：描述了第一个 Slave 的信息
  - offset：描述了该节点的更新进度
- master_replid：标识一个 Master，当 Master 启动或者是 Slave 晋升为 Master 时生成
- master_repl_offset：描述了 Master 处理写命令的进度
- master_replid2：标识了断开连接前上次 Master 的 master_replid
- second_repl_offset：描述了断开连接前上次 Master 处理写命令的进度
- slave_repl_offset：描述了 Slave 的处理写命令的进度

replid 和 offset 共同标识了一块数据集，若是两个节点的 replid 和 offset 都相等，则说明这两个节点的数据完全一样。

## 更改主从结构

通过 `slaveof` 命令可更改主从结构。

- Slave 执行 `slaveof no one`，表示主动脱离 Master，该节点成为一个独立的节点；
- Slave 执行 `slaveof ip port`，表示要成为该 redis-server 的 Slave。

但是要注意，在 redis-cli 中敲这些命令得到的效果只是暂时的，当 redis-server 重启时，会重新读取配置文件。

## 拓扑结构

**一主一从**

![image-20231127212409652](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127212409652.png)

此结构是最简单的复制拓扑结构，用于当 Master 发生故障宕机时，Slave 提供故障转移支持。当 Master 的写命令并发量较高且需要持久化时，可以选择 Master 不开启而 Slave 开启 AOF 日志。但当 Master 重启时需要先获取 Slave 的 AOF 日志。

**一主多从**

![image-20231127212422170](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127212422170.png)

此结构实现了“读写分离”，适用于读命令并发量高的场景，但是当写命令并发量高时，由于 Master 要重复的同步很多份的写命令，所以会严重影响 Master 的带宽。

**树状主从**

![image-20231127212506061](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231127212506061.png)

此结构是一主多从结构的优化版，当写命令并发量高时，Master 的带宽并不会很高，由中间层的 Slave 向底层的 Slave 同步命令。

## 主从复制的建立流程

![image-20231128211014984](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231128211014984.png)

（1）Slave 只保存 Master 的地址信息（ip、port）；

（2）Slave 尝试与 Master 建立 TCP 连接，若无法建立连接时，Slave 内部会无限不断尝试，直到用户主动取消；

（3）Slave 发送 ping 命令，检测 Master 在应用层是否能正常工作，当 Master 的 pong 回复超时时，Slave 会断开 TCP 连接；

（4）若 Master 配置了 requirepass 参数，则需要进行密码验证，Slave 通过配置 masterauth 参数来设置密码；

（5）Master 会选用全量复制或者部分复制进行对 Slave 的数据同步；

（6）主从复制连接建立完毕后，Master 会通过 TCP 连接不断的向 Slave 同步写命令（异步）。

### psync 同步数据集

Redis 使用 `psync` 命令进行主从节点的数据同步，数据同步分为全量复制和部分复制。

全量复制：一般用于主从节点初次建立连接时，或者是主从节点恢复连接但部分复制的条件不满足时。

部分复制：一般用于主从节点恢复连接时。

#### psync 的语法格式

```c++
psync replid offset
```

当 replid 为 ? ，offset 为 -1 时，就是在申请全量复制。

当 replid 为具体值，offset >= 0 时，就是在申请部分复制。

#### psync 的运行流程

![image-20231128194231683](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231128194231683.png)

（1）Slave 发送 psync 命令，默认值为 ？ -1；

（2）Master 发送响应，若为 +FULLRESYNC，则全量复制；

​				          若为 +CONTINUE，则部分复制；

​				          若为 -ERR，则 Master 版本过低，不支持 psync 命令，Slave 可尝试发送 sync 命令。

> psync 会在主从复制建立的过程中自动被调用；
>
> psync 不会阻塞主线程，sync 会阻塞主线程。

### 全量复制

当主从节点初次建立连接时，或者是主从节点恢复连接部分复制的条件不满足时进行全量复制。

#### 全量复制的流程

![image-20231128191026386](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231128191026386.png)

（1）Slave 发送 psync ? -1 命令，表示要与 Master 进行数据同步，且用全量复制的方式；

（2）Master 发送 +FULLRESYNC 响应，表示我已经收到了请求，等我准备数据；

（3）Slave 保存一些 Master 的相关信息；

（4）Master 调用 bgsave 命令生成 RDB 快照保存到磁盘（即使已经有 RDB 快照，此处也要重新生成）；

（5）Master 将新生成的 RDB 快照发送给 Slave；

（6）Master 将缓冲区的数据以 RDB 的形式发送给 Slave，Slave 将其追加到 RDB 快照中（Master 会将生成以及传输 RDB 快照期间内处理的写命令写到缓冲区中）；

（7）Slave 清空本地数据；

（8）Slave 加载 RDB 快照；

（9）如果 Slave 开启了 AOF 日志功能，则会在加载完 RDB 快照之后，执行 bgrewriteaof 命令，生成最新的 AOF 日志（因为在加载 RDB 快照的时候生成 AOF 日志中有可能数据冗余，所以需要再次整理生成新的 AOF 日志）。

#### 无磁盘复制（diskless）

通过以上流程可以发现，Master 会生成一份 RDB 快照保存到磁盘上然后通过网络进行发送，保存到磁盘上这一步骤对 Master 其实是一种负担，可以选择生成 RDB 快照后不保存到磁盘上而是直接通过网络发送。所以 Redis 从 2.8.18 版本开始支持无磁盘复制，可通过配置 `repl-diskless-sync yes` 选项将其打开。

### 部分复制

主从节点间的数据同步若每次都进行全量复制，开销是很大的，当 Slave 已经有 Master 的大部分数据时，就不需要进行全量复制，只需进行部分复制即可。

在主从节点因网络抖动等原因从断开连接到恢复连接的期间内，Master 会继续处理命令，而 Slave 只能干坐着。当恢复连接时，Slave 会发送当初保存的数据进度 offset，若 Master 通过检查，发现 offset 在当前的复制积压缓冲区中，就进行部分复制，反之进行全量复制。

#### 部分复制的流程

![image-20231128193042438](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231128193042438.png)

（1）主从连接出现网络中断时，若时间超出了 real-timeout，Master 就会认为 Slave 出现故障从而断开主从连接；

（2）断开主从连接之后，Master 仍会继续处理命令，但复制命令因为主从连接断开了而无法向 Slave 发送，所以会滞留在复制积压缓冲区内。

（3）Slave 恢复与 Master 的连接；

（4）Slave 发送 psync replid2 offset，表示想要与 Master 进行部分复制（replid2 保存着断开连接前的 Master 的 replid）；

（5）Master 接收到请求后，会先进行验证，然后根据 offset 去复制积压缓冲区内查找合适的数据，并返回 +CONTINUE 响应；

（6）Master 将数据同步给 Slave。

#### 复制积压缓冲区（repl-backlog-buffer）

Master 不仅会将处理过的命令发送给 Slave，还会将其写入到复制积压缓冲区内，用于部分复制和已发送命令丢失时的数据补救。

![image-20231128200138913](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231128200138913.png)

可通过 `info replication` 命令查看复制积压缓冲区：

![image-20231128200448828](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231128200448828.png)

- repl_backlog_active：复制积压缓冲区已开启
- repl_backlog_size：复制积压缓冲区的总大小
- repl_backlog_first_byte_offset：复制积压缓冲区内数据的起始偏移量
- repl_backlog_histlen： 复制挤压缓冲区内数据的总长度

复制挤压缓冲区内数据的可用偏移量范围：[repl_backlog_first_byte_offset, repl_backlog_first_byte_offset + repl_backlog_histlen]

复制挤压缓冲区其实就是一个由数组实现的环形队列。

### 实时复制

主从复制连接建立以后，Master 会将处理过的写命令通过 TCP 长连接实时发送给 Slave（异步），Slave 接收到写命令然后修改自身的数据从而保持与 Master 的数据一致。

这里的长连接需要通过心跳包的机制来维护连接状态（应用层实现的心跳包机制）：

- 主从节点彼此都有心跳检测机制，各自模拟成对方的客户端进行通信；
- Master 默认每隔 10 秒向 Slave 发送 ping 命令，来判断 Slave 的存活性与连接状态；
- Slave 默认每隔 1 秒 向 Master 发送 replconf ack {offset} 命令，向 Master 汇报自己的数据复制偏移量。	

若 Master 发现主从连接通信延迟超过 repl-timeout（默认 60 秒），则断开与 Slave 的主从连接。



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
