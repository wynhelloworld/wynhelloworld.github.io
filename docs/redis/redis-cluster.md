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

![image-20231204215648060](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231204215648060.png)

#### 一致性哈希

#### 哈希槽分区