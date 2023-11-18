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
