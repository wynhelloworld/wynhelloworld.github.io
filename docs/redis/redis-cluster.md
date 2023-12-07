# Redis 集群

Redis 主从复制提高了系统的可用性，但真正存储数据的还是 Master 和 Slave，当存储的数据量很大的时候，就容易接近 Master 或 Slave 所在机器的物理内存了，这时就很容易出现问题。解决单点内存不足问题的最简单解决办法就是引入更多的机器，Redis 集群就是在上述的思路之下引入了多组 Master / Slave，每一组 Master / Slave 存储数据全集的一部分。

如下图所示，假设数据全集为 1TB，引入了 3 组 Master / Slave，那么每组 Master / Slave 只需存储 1TB 的 1/3 即可。

![image-20231204213836867](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231204213836867.png)

上图中，各组之间的数据是不同的，各组之内的数据是相同的。每组被称为一个分片（Sharding）。

## 数据分片算法

Redis 集群（Redis Cluster）的核心思路就是用多组分片来存储整个数据全集的每个部分，那么下一个核心问题就是如何对数据全集进行分区。业界中有下述三个较为主流的方式。

#### 哈希求余

假设有 N 个分片，从 0 开始进行编号。

使用 hash 算法对 key 算出摘要，用摘要 % N，得到的结果即为分片编号。

![image-20231205200911946](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231205200911946.png)

优点：简单高效、数据分配均匀

缺点：在扩容的时候，数据迁移量太大。因为原来是 3 个分片，现在扩成了 4（或者更多）个分片，那么原来 3 个分片上的数据有很多都需要重新进行哈希求余寻找新的分片。

#### 一致性哈希

为了解决哈希求余扩容时的高开销，诞生了一致性哈希。

一致性哈希的原理：

1. 将 [0 - 2^31] 这个数据空间，映射到一个圆环上，数据按顺时针方向增长。

   ![image-20231205212428526](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231205212428526.png)

2. 假设现在有 3 个分片。

   ![image-20231205212841256](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231205212841256.png)

3. 然后，开始对 key 计算 hash 值，先找到 hash 值在该图上的位置，然后顺时针走，碰到几号分片，该 key 就属于几号分片。

   ![image-20231205213149619](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231205213149619.png)

   username 的 hash 值在 2 号分片和 0 号分片之间，然后顺时针走，碰到了 0 号分片，那么 username 属于 0 号分片。

优点：当进行扩容时，直接在环上新安排一个分片位置即可，比如在 0 号分片和 1 号分片中间添加一个 3 号分片，那么就只需要将 1 号分片上的一半的数据转移到 3 号分片上，1 号分片和 0 号分片不需要转移数据。所以，总的数据转移的开销并不是很高。

缺点：数据分散，有的分片数据多，有的分片数据少。

#### 哈希槽分区（Redis 使用）

为了解决数据转移的高开销和数据的分配不均匀，又诞生了哈希槽分区。

用 crc16 算法算出 key 的哈希摘要，然后用这个摘要去 % 16384，得到的结果是多少，该 key 就属于哪个槽位。也就是说，哈希槽分区算法，一共有 16384 个槽位（2^14，2KB），然后将所有的 key 分配到这 16384 个槽位上。

然后再根据分片的数量，（不严格）均匀的分配这 16384 个槽位。

假设有三个分片的话：

0 号分片占有 [0, 5461] 槽位

1 号分片占有[5462, 10923] 槽位

2 号分片占有[10924, 16383] 槽位

每个节点用位图来表示自己所拥有哪些槽位。

当需要进行扩容的时候，每号分片只需拿出一点自己的槽位分配给新分片即可。（也就是说，槽位的分配不一定是连续的）

**Redis 集群最多能有 16384 个分片吗？**

每个分片并不是一个服务器，若每个分片只占有一个槽位，那么这个集群的复杂度就极高极高，这样的集群的可用性是很低的。所以 Redis 的作者建议分片数量不超过 1000。

**为什么是 16384 个槽位？**

16384 个 bit，正好是 2KB。Redis 节点之间通过心跳包来进行交互，若用比 2KB 更大的 4KB、8KB 等，会增加网络消耗。

况且 Redis 集群的分片数量一般不建议超过 1000，而 2KB 已经能够满足 1000 个分片了。

## 搭建集群

基于 docker 在 1 台服务器上搭建集群，拓扑结构如下：

![image-20231207210825254](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207210825254.png)

**创建集群工作目录**

```c++
mkdir redis-cluster
```

**进入 redis-cluster 目录，创建一个脚本，用来生成 11 个 Redis 节点的目录和配置文件（9 个用来搭建上述拓扑结构，2 个用来进行扩容测试）**

```shell
touch generate.sh
```

**编辑脚本内容如下：**

```shell
for port in $(seq 1 9); \
do \
mkdir -p redis${port}/
touch redis${port}/redis.conf
cat << EOF > redis${port}/redis.conf
port 6379
bind 0.0.0.0
protected-mode no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.30.0.10${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
EOF
done

# cluster-announce-ip .
for port in $(seq 10 11); \
do \
mkdir -p redis${port}/
touch redis${port}/redis.conf
cat << EOF > redis${port}/redis.conf
port 6379
bind 0.0.0.0
protected-mode no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.30.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
EOF
done
```

**执行脚本**

```shell
bash generate.sh
```

**创建 docker-compose.yml **

```shell
touch docker-compose.yml
```

**编辑内容如下：**

```shell
version: '3.7' 
networks:
  mynet: 
    ipam:
      config:
        - subnet: 172.30.0.0/24
        
services: 
  redis1:
    image: 'redis:5.0.9' 
    container_name: redis1 
    restart: always 
    volumes:
      - ./redis1/:/etc/redis/
    ports:
      - 6371:6379
      - 16371:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.101

  redis2:
    image: 'redis:5.0.9' 
    container_name: redis2
    restart: always 
    volumes:
      - ./redis2/:/etc/redis/
    ports:
      - 6372:6379
      - 16372:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.102

  redis3:
    image: 'redis:5.0.9' 
    container_name: redis3
    restart: always 
    volumes:
      - ./redis3/:/etc/redis/
    ports:
      - 6373:6379
      - 16373:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.103

  redis4:
    image: 'redis:5.0.9' 
    container_name: redis4
    restart: always 
    volumes:
      - ./redis4/:/etc/redis/
    ports:
      - 6374:6379
      - 16374:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.104

  redis5:
    image: 'redis:5.0.9' 
    container_name: redis5
    restart: always 
    volumes:
      - ./redis5/:/etc/redis/
    ports:
      - 6375:6379
      - 16375:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.105

  redis6:
    image: 'redis:5.0.9' 
    container_name: redis6
    restart: always 
    volumes:
      - ./redis6/:/etc/redis/
    ports:
      - 6376:6379
      - 16376:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.106

  redis7:
    image: 'redis:5.0.9' 
    container_name: redis7
    restart: always 
    volumes:
      - ./redis7/:/etc/redis/
    ports:
      - 6377:6379
      - 16377:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.107

  redis8:
    image: 'redis:5.0.9' 
    container_name: redis8
    restart: always 
    volumes:
      - ./redis8/:/etc/redis/
    ports:
      - 6378:6379
      - 16378:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.108

  redis9:
    image: 'redis:5.0.9' 
    container_name: redis9
    restart: always 
    volumes:
      - ./redis9/:/etc/redis/
    ports:
      - 6379:6379
      - 16379:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.109

  redis10:
    image: 'redis:5.0.9' 
    container_name: redis10
    restart: always 
    volumes:
      - ./redis10/:/etc/redis/
    ports:
      - 6380:6379
      - 16380:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.110

  redis11:
    image: 'redis:5.0.9' 
    container_name: redis11
    restart: always 
    volumes:
      - ./redis11/:/etc/redis/
    ports:
      - 6381:6379
      - 16381:16379
    command:
      redis-server /etc/redis/redis.conf
    networks: 
      mynet:
        ipv4_address: 172.30.0.111
```

**启动容器**

```shell
docker-compose up -d
```

如下图所示，即为启动成功。

![image-20231207212011778](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207212011778.png)

**构建集群关系**

```shell
redis-cli --cluster create 172.30.0.101:6379 172.30.0.102:6379 172.30.0.103:6379 172.30.0.104:6379 172.30.0.105:6379 172.30.0.106:6379 172.30.0.107:6379 172.30.0.108:6379 172.30.0.109:6379 --cluster-replicas 2
```

出现下图所示情况后，输入 yes，按回车。

![image-20231207212259425](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207212259425.png)

如下图所示，即为集群成功搭建完成。

![image-20231207212358093](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207212358093.png)

最终搭建了一个如下图所示的集群：

![image-20231207212944034](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207212944034.png)

实际的搭建情况和一开始设定的不一样，但很正常，因为每个节点之间都是平等的，Master 和 Slave 全由 Redis 自己分配。

#### **有两种方式登录客户端**

```c++
redis-cli -p 6371
```

```c++
redis-cli -h 172.30.0.101 -p 6379
```

以上两种方式登陆的是同一个客户端。

#### **在客户端里查看集群关系**

```shell
cluster notes
```

如下图所示：

![image-20231207213619194](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207213619194.png)

#### 客户端自动切换

先看图

![image-20231207214041208](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207214041208.png)

在 101 这台机器上设置 k1，显示无法设置。这是因为 k1 经过哈希算法求出的摘要 % 16384 后得出的槽位属于 103 分片，而当前客户端是 101 分片，所以无法在 101 分片上去设置属于 103 分片的值。

为了解决上述问题，可以在登陆客户端的时候添加选项 `-c` ，如：

```c++
redis-cli -h 172.30.0.101 -p 6379 -c
```

然后设置 k1

![image-20231207214755595](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207214755595.png)

会发现客户端自动从 101 变成了 103，然后再设置一个 k2

![image-20231207214910783](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231207214910783.png)

客户端又从 103 变成了 101。





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