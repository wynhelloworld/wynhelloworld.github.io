# Redis 作为缓存

用户的数据一般存储于数据库，而数据库中的数据又存储于磁盘上，众所周知，磁盘的读写速度算是计算机中最慢的了。

当有大量请求都打到数据库中时，由于数据库极慢的处理速度，很容易导致数据库崩溃，所以通常会用 Redis 作为数据库的缓存。

因为[二八法则](https://zh.wikipedia.org/wiki/%E5%B8%95%E7%B4%AF%E6%89%98%E6%B3%95%E5%88%99)，所以用 Redis 缓存少量的数据就能够处理绝大多数的请求。

![image-20231212221532318](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231212221532318.png)

## Redis 缓存更新策略

### 定期生成

Redis 会把用户访问的数据用日志的形式记录下来，然后按照策略（每天、每周、每月）定期更新。

当需要进行更新时，就对日志进行统计，按照 Key 的访问频率降序排序，前 20% 就是热点数据。

但该方法的实时性不高，比如春节期间的晚上九点，搜索某个小品的频率是很高的，但此时关于该小品的数据并不作为热点数据缓存在 Redis 中，所以会有大量的请求打到数据库上，容易导致数据库崩溃。

### 实时生成

若 Redis 中有用户要访问的数据，那么就直接返回给用户，反之 Redis 就去数据库访问该数据，然后把该数据缓存在 Redis 中，再返回给用户。

但这样时间一长，容易导致 Redis 内存紧张，从而导致 Redis 崩溃。所以，会有一些 Key 的淘汰策略来平衡 Redis 的内存占用情况。

> Redis 内存紧张，并不一定是指 Redis 所在的服务器的内存紧张，Redis 也可以设置 Redis 服务的内存容量大小。

#### Key 淘汰策略

- **FIFO（First In First Out）先存储的先淘汰**

- **LRU（Least Recently Used）淘汰最久未使用的**

- **LFU（Least Frequently Used）淘汰访问次数最少的**

- **Random 随机淘汰**

#### Redis 内置的 Key 淘汰策略

- `volatile-lru`：当内存不足以容纳新写入的数据时，从设置了过期时间的 key 中采用 LRU 算法进行淘汰。
- `allkeys-lru`：当内存不足以容纳新写入的数据时，从所有的 key 中采用 LRU 算法进行淘汰。
- `volatile-lfu`：当内存不足以容纳新写入的数据时，从设置了过期时间的 key 中采用 LFU 算法进行淘汰。（4.0 引入）
- `allkeys-lfu`：当内存不足以容纳新写入的数据时，从所有的 key 中采用 LFU 算法进行淘汰。（4.0 引入）
- `volatile-random`：当内存不足以容纳新写入的数据时，从设置了过期时间的 key 中采用 Random 算法进行淘汰。
- `allkeys-random`：当内存不足以容纳新写入的数据时，从所有的 key 中采用 Random 算法进行淘汰。
- `volatile-ttl`：当内存不足以容纳新写入的数据时，从设置了过期时间的 key 中根据过期时间进行淘汰（先过期的优先淘汰）。
- `noeviction`：当内存不足以容纳新写入的数据时，执行写入操作报错。（默认策略）

## 关于缓存失效，导致数据库崩溃的四种情况

### 缓存预热（Cache preheating）

当 Redis 刚启动或者大批 key 失效时，由于 Redis 中没有什么数据，所以会有很多请求打到数据库上，容易导致数据库崩溃。

此时，可以把准备好的热点数据导入 Redis 中，使 Redis 尽早成长起来。

准备好的热点数据可以通过离线、统计等手段得到。

### 缓存穿透（Cache penetration）

若 Redis 和数据库中都没有用户要访问的数据，并且之后还有大量的请求访问该数据，那么这些请求都会打到数据库上，容易导致数据库崩溃。

#### 为什么会有大量的请求访问 Redis 和数据库中都没有的数据？

- 业务设计不合理，没有进行必要的校验环节。
- 误删除了 Redis 和数据库中的某些数据。
- 黑客攻击。

#### 如何解决？

- 在前端进行校验（比如合理的手机号）。
- 当访问的某个 key 在 Redis 和数据库中都不存在时，可以将该 key 的值设置成 `“”` 存储在 Redis 中。
- 使用布隆过滤器来判断某个 key 在 Redis 和数据库中是否不存在。

### 缓存雪崩（Cache avalanche）

短时间内 Redis 上大量的 key 失效，导致大量的请求打到数据库上，容易导致数据库崩溃。

#### 为什么会存在段时间内大量 key 失效？

- Redis 挂了。
- Redis 上的设置了过期时间的 key 同时过期被淘汰了。

#### 如何解决？

- 部署高可用的 Redis 集群，完善监控报警体系。
- 不给 key 设置过期时间，或者给 key 设置过期时间时添加随机因子。

### 缓存击穿（Cache breakdown）

是缓存雪崩的一种特殊情况，短时间内 Redis 上大量的热点数据失效，导致大量的请求打到数据库上，容易导致数据库崩溃。

#### 如何解决？

- 针对热点数据，不设置过期时间。
- 进行必要的服务降级。（例如访问数据库时采用分布式锁，限制同时请求数据库的并发数）



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
