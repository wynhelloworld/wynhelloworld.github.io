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

优点：当进行扩容时，直接在环上新安排一个分片位置即可，比如在 0 号分片和 1 号分片中间添加一个 3 号分片，那么就只需要将 1 号分片上的一半的数据转移到 3 号分片上，1 号分片和 0 号分片不需要转移数据。

缺点：数据分散，有的分片数据多，有的分片数据少。

#### 哈希槽分区（Redis 使用）





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