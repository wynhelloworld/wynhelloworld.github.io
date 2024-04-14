# Redis 持久化

Redis 是内存数据库，一但发生断电等事故时，所有的数据都会丢失，所以 Redis 提供了两种持久化方案：RDB 快照和 AOF 日志。

持久化是指将数据写入持久化存储，例如固态硬盘 (SSD) 。

## RDB 快照

RDB 快照就是将 Redis 服务器某个瞬间的状态记录下来保存到硬盘的过程。

#### 触发机制

- 手动触发

  - `save`：在主线程中生成快照，会阻塞主线程。
  - `bgsave`：在主线程 fork 的子进程中生成快照，仅在 fork 的时候阻塞主线程，由于 fork 时采用写时拷贝技术（Copy-On-Write），所以阻塞时间很短。
  
- 自动触发

  自动触发默认开启，可在 `redis.conf` 配置文件中对其进行配置，默认会提供以下配置：

  ```shell
  save 900 1
  save 300 10
  save 60 10000
  ```
  
- 除了以上两种官方说明的触发机制以外，当 Redis 服务器正常关闭 (重启) 时，也会自动的进行 RDB 快照，当使用 `kill -9` 杀死 Redis 服务器进程时，Redis 服务器会来不及进行 RDB 快照。

#### 执行流程

![image-20231121171630225](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231121171630225.png)

#### 文件处理

- RDB 文件的文件名和存储目录可在 redis.conf 配置文件中对其进行配置，默认会提供以下配置： 

  ```shell
  dbfilename dump.rdb
  dir /var/lib/redis
  ```
  
- RDB 文件是默认被 LZF 算法进行压缩处理过的二进制文件，压缩后的文件大小远小于内存大小，是默认被开启的。可在 redis.conf 配置文件中进行配置，关闭压缩算法，默认配置如下：

  ```shell
  rdbcompression yes
  ```
  
  虽然压缩会消耗 CPU 资源，但会更方便的进行存储与传输，所以建议开启。
  
- 每次进行 RDB 持久化时，都会重新创建 dump.rdb 文件覆盖掉旧文件（inode 会变化）。

- 当 RDB 文件发生损坏时，会影响 Redis 服务器的启动（主要是 RDB 文件前面的部分损坏时），当 RDB 文件后面的部分损坏时，可能不会影响 Redis 服务器的启动，但会影响数据。所以在启动服务器时，可以用 Redis 提供的工具 `redis-check-dump` 对 RDB 文件进行校验。

#### 优缺点

- RDB 文件是一个紧凑压缩的二进制文件，是 Redis 服务器在某个时间点上的快照，适合备份、全量复制等场景。
- RDB 文件是特定格式的二进制文件，由于 Redis 服务器的不断更新，所以可能存在版本兼容性问题。
- RDB 快照方式恢复数据的速度要远快于 AOF 日志方式。
- RDB 快照方式没办法做到实时持久化/秒级持久化。
- RDB 快照方式每次都会 fork 子进程，频繁执行时，成本较高。

## AOF 日志

AOF 日志就是把 Redis 服务器所处理过的每一条**写命令**，以文本的形式保存到硬盘的过程。

当 Redis 服务器启动时，读取这个 AOF 日志，就会重现之前执行过的每一条写命令，以达到恢复数据的目的。

AOF 日志默认是关闭的，可通过修改配置文件将其打开（将 no 修改成 yes），默认配置如下：

```shell
appendonly no
```

#### 日志写回硬盘策略

AOF 日志共有三种写回硬盘的策略：

- `always`：每成功处理完一条写命令后，先将命令写入 aof_buf 缓冲区中，然后调用 write 函数将缓冲区刷新到 AOF 文件中，再调用 fsync 函数将 AOF 文件数据写回到硬盘。
- `everysec`：每成功处理完一条写命令后，先将命令写入 aof_buf 缓冲区中，然后只调用 write 函数，每隔 1 sec 由异步线程调用 fsync 函数将 AOF 文件数据写回到硬盘。
- `no`：每成功处理完一条写命令后，先将命令写入 aof_buf 缓冲区中，然后只调用 write 函数，不负责调用 fsync 函数，由 OS 负责调用 fsync 函数。

一、三两种策略相比策略二较为极端，但这些策略不分好坏，选用哪种策略，应根据实际的应用场景而定。

Redis 服务器默认采用策略二，配置如下：

```shell
# appendfsync always
appendfsync everysec
# appendfsync no
```

#### 重写机制

由于 Redis 服务器会不断处理命令，所以 AOF 日志文件也会不断的变大，而较大的 AOF 文件会导致下次 Redis 服务器重启时耗时较长，并且较大的 AOF 文件中会出现很多的命令冗余（AOF 会记录每一条命令，而 AOF 日志应记录最新的服务器状态，所以 AOF 文件中会出现很多的中间命令是无效的），所以当 AOF 文件的时长/大小达到一定程度后，会根据当前 Redis 服务器的状态，重写 AOF 文件。 

AOF 文件的重写分为手动触发和自动触发:

- 手动触发：执行 `bgrewriteaof` 命令。

- 自动触发：当开启 AOF 日志机制后，自动触发就已生效，默认配置如下：

  ```shell
  auto-aof-rewrite-percentage 100
  auto-aof-rewrite-min-size 64mb
  ```
  
  - `auto-aof-rewrite-percentage`： 表示当前 AOF 占用大小相比较上次重写增加的比例。
  - `auto-aof-rewrite-min-size`：表示触发重写时 AOF 的最小文件大小为 64MB。

#### AOF 的重写流程

![image-20231120213009471](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231120213009471.png)

**为什么 fork 之后，父进程（Redis 主线程）仍然会往缓冲区里写呢？**

如果子进程出现问题了然后挂了，那么此时父进程也没往缓冲区里写，于是就没往旧 AOF 文件里写，此时就会造成数据丢失。

**为什么 fork 之后，父进程会多往一个 `aof_rewrite_buf` 缓冲区里写呢？**

因为父子进程是相互隔离的，在子进程进行 AOF 操作期间，父进程处理的所有命令，子进程是感知不到的，所以需要父进程专门再为子进程 (新 AOF 文件) 准备一份缓冲区，当子进程发送信号通知父进程 AOF 已处理好时，父进程会调用相应的处理函数，将 aof_rewrite_buf 里的数据刷新到新 AOF 文件中。

## Redis 服务器启动时的数据恢复流程

![image-20231120214923758](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231120214923758.png)

## 混合持久化

AOF 本来就是按照文本的形式记录命令的，以恢复数据的。但随之时间与大小的增长，后续的成本还是挺高的，所以 Redis 官方又引入了混合持久化，结合了 RDB 快照和 AOF 日志的特点。当 AOF 日志进行重写时，以 RDB 快照的方式记录下当前服务器的状态并存入 AOF 文件中（依旧是二进制），当重写完毕后，还是用文本的形式记录命令。混合持久化机制默认是开启的，可通过修改配置文件进行关闭，默认配置如下：

```shell 
aof-use-rdb-preamble yes
```



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
