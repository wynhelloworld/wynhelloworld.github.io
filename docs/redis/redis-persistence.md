# 持久化

持久化是指将数据写入持久化存储, 例如固态硬盘 (SSD) 

Redis 支持 RDB 快照和 AOF 日志两种持久化机制

## RDB 快照

RDB 快照就是把 Redis 服务器内存中的所有数据生成快照保存到硬盘的过程

#### RDB 触发机制

- 手动触发

  手动触发有两种命令

  - `save` : 在主线程中生成快照, 会阻塞主线程
  - `bgsave` : 在主线程 fork 的子进程中生成快照, 仅在 fork 的时候阻塞主线程, 由于 fork 时采用写时拷贝技术 (Copy-On-Write), 所以阻塞时间很短

- 自动触发

  自动触发需要在 `redis.conf` 配置文件中进行配置, 默认会提供以下配置:

  ```shell
  ################################ SNAPSHOTTING  ################################
  #
  # Save the DB on disk:
  #
  #   save <seconds> <changes>
  #
  #   Will save the DB if both the given number of seconds and the given
  #   number of write operations against the DB occurred.
  #
  #   In the example below the behaviour will be to save:
  #   after 900 sec (15 min) if at least 1 key changed
  #   after 300 sec (5 min) if at least 10 keys changed
  #   after 60 sec if at least 10000 keys changed
  #
  #   Note: you can disable saving completely by commenting out all "save" lines.
  #
  #   It is also possible to remove all the previously configured save
  #   points by adding a save directive with a single empty string argument
  #   like in the following example:
  #
  #   save ""
  
  save 900 1
  save 300 10
  save 60 10000
  ```

- 除了以上两种官方说明的触发机制以外, 当 Redis 服务器正常关闭 (重启) 时, 也会自动的进行 RDB 快照, 当使用 `kill -9` 杀死 Redis 服务器进程时, Redis 服务器会来不及进行 RDB 快照

#### RDB 文件相关

- RDB 文件默认保存在 `/var/lib/redis/` 目录中, 文件名默认为 `dump.rdb`. 可在 `redis.confg` 配置文件中进行配置. 

  ```shell
  # The filename where to dump the DB
  dbfilename dump.rdb
  
  # The working directory.
  #
  # The DB will be written inside this directory, with the filename specified
  # above using the 'dbfilename' configuration directive.
  #
  # The Append Only File will also be created inside this directory.
  #
  # Note that you must specify a directory here, not a file name.
  dir /var/lib/redis
  ```

- RDB 文件是二进制文件, 默认被 LZF 算法进行压缩处理过, 压缩后的文件大小远小于内存大小, 默认开启. 可通过 `redis.confg` 配置文件进行配置. 

  ```shell
  # Compress string objects using LZF when dump .rdb databases?
  # For default that's set to 'yes' as it's almost always a win.
  # If you want to save some CPU in the saving child set it to 'no' but
  # the dataset will likely be bigger if you have compressible values or keys.
  rdbcompression yes
  ```

- 每次进行 RDB 持久化时, 都会重新创建 `dump.rdb` 文件(inode 会变化). 

- 当 RDB 文件发生损坏时, 会影响 Redis 服务器的启动 (主要是 RDB 文件前面的部分损坏时), 当 RDB 文件后面的部分损坏时, 可能不会影响 Redis 服务器的启动, 但会影响数据. 所以在启动服务器时, 可以用 Redis 提供的工具 `redis-check-dump` 对 RDB 文件进行校验.

#### RDB 快照的优缺点

- RDB 文件是一个紧凑压缩的二进制文件, 代表 Redis 在某个时间点上的快照. 适合备份、全量复制等场景
- RDB 文件是特定格式的二进制文件, 可能存在版本兼容性问题
- RDB 方式恢复数据的速度要远快于 AOF 方式
- RDB 方式没办法做到实时持久化/秒级持久化
- RDB 方式每次都会 `fork` 子进程, 频繁执行时, 成本较高

## AOF 日志

AOF 日志就是把 Redis 服务器所处理过的每一条**写命令**, 以文本的形式保存到硬盘的过程

当 Redis 服务器启动时, 读取这个 AOF 日志, 就会重现之前执行过的每一条写命令, 以达到恢复数据的目的

#### AOF 触发时机

AOF 日志默认是关闭的, 默认配置如下:

```shell
# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no
```

要开启 AOF 日志, 仅需将 no 修改成 yes 即可.

#### AOF 默认文件名

```shell
# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"
```

#### AOF 写回策略

Redis 服务器会将成功处理完的写命令先写入到缓冲区中, 再将缓冲区写入到 AOF 日志中, 但根据缓冲区写入到 AOF 日志的时机不同, 分为了三种写回策略

- `always`: 每处理完一条成功写命令, 就立即将缓冲区写入到 AOF 日志中
- `everysec`: 每隔 1 sec, 就将缓冲区写入到 AOF 日志中一次
- `no`: Redis 服务器不进行处理, 由 OS 权衡利弊, 进行缓冲区到 AOF 日志的写入

前两种策略, 都是在 Redis 服务器主线程中执行的, 会消耗 Redis 服务器的性能, 第三种策略不会. 这三种策略使得数据完整性与服务器性能呈现反比的状态, `always` 策略使得数据近似 100% 完整, 但服务器性能消耗较大, `everysec` 策略每秒写入一次, 在断电/宕机的时候会导致前 1 秒的数据丢失, 服务器性能消耗较小, `no` 策略的写入时机不定, 在断电/宕机时可能导致较多的数据丢失, 但不消耗服务器性能. 综合而言, `always` 与 `no` 策略较为极端, `everysec` 适中, 但最终采用什么策略应根据应用场景而定.

当开启 AOF 日志时, 默认配置如下:

```shell
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no
```

#### AOF 的重写机制

由于写命令的增多, AOF 日志文件也会不断变大, 而较大的 AOF 文件会导致下次 Redis 服务器重启时耗时较长, 并且较大的 AOF 文件中会出现很多的**命令冗余**, 所以当 AOF 文件的时长/大小达到一定程度后, 会根据当前 Redis 服务器的状态, 重写 AOF 文件. 

AOF 文件的重写分为手动触发和自动触发:

- 手动触发: 执行 `bgrewriteaof` 命令

- 自动触发: 当开启 AOF 日志机制后, 自动触发就已生效, 默认配置如下

  ```shell
  # Automatic rewrite of the append only file.
  # Redis is able to automatically rewrite the log file implicitly calling
  # BGREWRITEAOF when the AOF log size grows by the specified percentage.
  #
  # This is how it works: Redis remembers the size of the AOF file after the
  # latest rewrite (if no rewrite has happened since the restart, the size of
  # the AOF at startup is used).
  #
  # This base size is compared to the current size. If the current size is
  # bigger than the specified percentage, the rewrite is triggered. Also
  # you need to specify a minimal size for the AOF file to be rewritten, this
  # is useful to avoid rewriting the AOF file even if the percentage increase
  # is reached but it is still pretty small.
  #
  # Specify a percentage of zero in order to disable the automatic AOF
  # rewrite feature.
  
  auto-aof-rewrite-percentage 100
  auto-aof-rewrite-min-size 64mb
  ```

  - `auto-aof-rewrite-percentage`: 表示触发重写时 AOF 的最小文件大小为 64MB
  - `auto-aof-rewrite-min-size`: 表示当前 AOF 占用大小相比较上次重写增加的比例

#### AOF 的重写流程

![image-20231120213009471](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231120213009471.png)

**为什么 fork 之后, 父进程 (Redis 主线程) 仍然会往缓冲区里写呢?**

如果子进程出现问题了然后挂了, 那么此时父进程也没忘缓冲区里写, 没往旧 AOF 文件里写, 此时就会造成数据丢失

**为什么 fork 之后, 父进程会多往一个 `aof_rewrite_buf` 缓冲区里写呢?**

因为父子进程是相互隔离的, 在子进程进行 AOF 操作期间, 父进程处理的所有命令, 子进程是感知不到的, 所以需要父进程专门再为子进程 (新 AOF 文件) 准备一份缓冲区, 当子进程发送信号通知父进程 AOF 以处理好时, 父进程会调用相应的处理函数, 将 `aof_rewrite_buf` 里的数据追加到新 AOF 文件之后

#### Redis 服务器启动时的数据恢复

当 RDB 快照和 AOF 日志都开启时, RDB 快照就不生效了, Redis 服务器只读取 AOF 日志. 流程如下:

![image-20231120214923758](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231120214923758.png)

## 混合持久化

AOF 日志虽然会在重写时, 不断压缩大小, 但以文本的形式存储每一条有效写命令, 还是会占空间, 在读取时也消耗时间, 所以 Redis 官方又引入了混合持久化, AOF 日志在每次重写时, 会将 Redis 服务器的状态以 RDB 快照的方式将二进制数据写入 AOF 日志中, 启动之后, 处理的每一条命令仍以文本的形式追加到 AOF 日志中. 混合持久化默认开启, 可通过配置文件关闭, 默认配置如下:

```shell 
# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
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
