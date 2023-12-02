# Redis 哨兵

在 Redis 主从复制模式下，如果 Master 发生故障而停止服务时，需要进行主从切换：从众多 Slave 中挑选出一个当作 Master（执行 slaveof no one 命令），并且要重新配置其它的 Slave 成为新的 Master 的 Slave（执行 slaveof master_ip master_port 命令）。

## 人工进行主从切换

若主从切换是人工进行的，那么通常需要搭配一个监控程序和一个报警程序。一但监控程序发现主节点发生故障而停止服务时，就让报警程序通过短信、电话、微信、邮件等等手段来通知程序员，然后程序员就赶快上线去修补故障，若能让主节点迅速恢复那就恢复，若不能迅速恢复就进行主从切换。但是这通常要花费很长时间（少则半个小时吧），而这是绝对不被容许的，所以 Redis 提出了**哨兵机制**去解决人工进行主从切换的问题。

## 哨兵机制自动进行主从切换

Redis Sentinel 是一个独立的进程。Sentinel 节点会监控所有的 Master、Slave 和其余 Sentinel，当发现 Master 出故障时，就自动进行主从切换实现故障转移。

#### 使用 Docker 搭建 3 Sentinel 1Master 2 Slave 架构

![image-20231130215540198](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231130215540198.png)

创建数据节点的工作目录

```shell
mkdir redis-data
```

进入 redis-data 目录，并创建 docker-compose.yml 配置文件

```shell
touch docker-compose.yml
```

编辑 docker-compose.yml 配置文件

```shell
version: '3.7'
services:
  master:
    image: 'redis:5.0.9'
    container_name: redis-master
    restart: always
    command: redis-server --appendonly yes
    ports:
      - 6379:6379
  slave1:
    image: 'redis:5.0.9'
    container_name: redis-slave1
    restart: always
    command: redis-server --appendonly yes --slaveof redis-master 6379
    ports:
      - 6380:6379
  slave2:
    image: 'redis:5.0.9'
    container_name: redis-slave2
    restart: always
    command: redis-server --appendonly yes --slaveof redis-master 6379
    ports:
      - 6381:6379
```

创建哨兵节点的工作目录

```shell
mkdir redis-sentinel
```

进入 redis-sentinel 目录，并创建 docker-compose.yml 配置文件

```shell
touch docker-compose.yml
```

编辑 docker-compose.yml 配置文件

```shell
version: '3.7'
services:
  sentinel1:
    image: 'redis:5.0.9'
    container_name: redis-sentinel-1
    restart: always
    command: redis-sentinel /etc/redis/sentinel.conf
    volumes:
      - ./sentinel1.conf:/etc/redis/sentinel.conf 
    ports:
      - 26379:26379
  sentinel2:
    image: 'redis:5.0.9'
    container_name: redis-sentinel-2
    restart: always
    command: redis-sentinel /etc/redis/sentinel.conf
    volumes:
      - ./sentinel2.conf:/etc/redis/sentinel.conf
    ports:
      - 26380:26379 
  sentinel3:
    image: 'redis:5.0.9'
    container_name: redis-sentinel-3
    restart: always
    command: redis-sentinel /etc/redis/sentinel.conf
    volumes:
      - ./sentinel3.conf:/etc/redis/sentinel.conf
    ports:
      - 26381:26379
networks:
  default:
    external:
      name: redis-data_default
```

创建 3 个 sentinel 配置文件

```c++
mkdir sentinel1.conf
mkdir sentinel2.conf
mkdir sentinel3.conf
```

将这 3 个配置文件都编辑上相同的内容

```shell
bind 0.0.0.0
port 26379
sentinel monitor redis-master 172.18.0.2 6379 2
sentinel down-after-milliseconds redis-master 1000
```

先进入 redis-data 目录启动数据节点

```shell
docker-compose up -d
```

再进入 redis-sentinel 目录启动哨兵节点

```shell
docker-compose up -d
```

####  主从切换的具体流程

先执行 `docker stop redis-master` 停掉 Master，营造 Master 因故障宕机的场景，从而进行主从切换。

![image-20231202185658963](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231202185658963.png)

1. 默认情况下，Sentinel 每隔 1s 会向所有 Master、Slave 发送一个 Ping 命令，若有节点没有在 `down-after-milliseconds` 时间内回复 Pong，则 Sentinel 判定该节点为主观下线（sdown)。
2. Sentinel 判定 Master 为主观下线后，为了验证自己的判断是否正确，会让其它 Sentinel 进行投票，当票数 >= `quorum` 时，所有的 Sentinel 也就会判定 Master 为客观下线（odown）。
3. Master 被判定为客观下线后，所有监控 Master 的 Sentinel 会进行协商，选举出一个 Leader 进行故障转移操作。由于在相近时间内可能会有多个 Sentinel 判定 Master 为客观下线，所以可能会有多个 Sentinel 同时竞选 Leader，这些 Sentinel 会进行拉票和投票，谁拉到的票数先 >= 哨兵个数 / 2 + 1 谁就是 Leader。
4. 故障转移的步骤如下：
   1. 选择新 Master：
      1. Leader 会先过滤掉网络连接状态不好的 Slave。
      2. 选择优先级最低的 Slave。若优先级都相同则继续。
      3. 选择 offset 最大的 Slave。若 offset 都相同则继续。
      4. 选择 runID 最小的 Slave。
   2. 配置新 Master：
      1. Leader 会让新 Master 执行 slave no one 命令，脱离掉原来的 Master，成为一个独立节点。
      2. Leader 每隔 1s 向新 Master 发送一次 INFO 命令，观察收到的信息。
      3. 当从收到的信息观察到 role: master 后，执行后面的步骤。
   3. Sentinel 会让其它的 Slave 执行 slaveof <新 Master IP> <新 Master Port>。





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