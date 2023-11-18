# 初识 Redis

[redis](https://redis.io/)目前有十种数据类型, 常见的有五种: String, List, Hash, Set, Zset

[单击](https://redis.io/commands/)查看 redis 所有命令

redis 存储数据都是用 key - value 的形式存储的, 而 redis 所说的数据类型都是在说 value, key 永远都是 String

redis 是典型的 C/S 设计程序, 客户端每敲一个命令, 都会进行一次网络传输. 所以, redis 有很多命令支持同时操作多个 key

redis 的单线程指的是: 所有的命令请求是在一个线程中进行处理的. 所以, 要尽量避免使用时间复杂度较高的命令, 如 `KEYS *`

redis 有很多时间复杂度为 O(N) 的命令, 但 N 有时指数据库中 key 的数量, 有时指命令后跟的参数 key 的数量, 后者常可视为 O(1)

## redis 的 key 的删除策略有哪些?

- 过期删除
  - 惰性删除: 当 key 的过期时间已经到时不进行删除, 而是当下一次访问该 key 时才进行删除, 并告诉调用者该 key 不存在
  - 定期删除: redis 内核会定期抽取一部分 key 进行检测, 删除已经到了过期时间的 key. (抽取、检测的速度要很快, 否则会阻塞线程)
- 内存淘汰

## redis 内部编码

redis 的每种数据类型都有自己的内部编码, 而且是多种, redis 会根据 value 的规模、类型自行挑选内部编码进行底层实现. 内部编码对于用户而言是透明的, 用户只能感知到上层的五种数据结构, 而无法感知到内部编码. 可通过 `object encoding` 命令查看 key 的内部编码.

## String

#### 内部编码

| 内部编码 | 解释                                                         |
| -------- | ------------------------------------------------------------ |
| int      | 64 位的整数 (相当于 C++ 中的 long long)                      |
| embstr   | 压缩字符串, 适用于表示比较短的字符串                         |
| raw      | 普通字符串, 适用于表示比较长的字符串 (相当于 C++ 中的 std::string) |

**NOTE: **浮点数不是以 int 的形式存储的, 而浮点数一般又不会太长, 所以浮点数一般是以 embstr 编码存储, 除非浮点数特别长用 raw 编码

#### 应用场景

- 缓存对象

  - 当应用服务器访问数据的时候, 先访问 redis, 若 redis 上存在该数据就返回, 若 redis 上不存在该数据, 就将请求转发到数据库 (如 mysql) 上, 数据库再将数据保存到 redis 上, redis 再向应用服务器返回该数据. 所以 redis 作为缓存的时候, 常用来存储**热点数据**. 但随着时间的推移, redis 上的缓存会越来越多, 进而降低效率, 所以此时有几种策略: 写入缓存的时候添加过期时间, 内存淘汰

    ![image-20231115212346129](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231115212346129.png)

- 常规计数

  - redis 处理命令是单线程的, 所以 String (int) 很适合进行计数场景, 如计算点赞数量、转发数量. 但 redis 不擅长数据统计, 比如统计一下某网站播放量前 100 的视频, 用 redis 搞就很麻烦, 但用 mysql 搞, 一条 sql 语句就 OK 了

- 手机验证码

  - 通过设置过期时间可以设置 1min、2min 等短信验证码功能, 还可以通过设置 nx 选项, 限制期限内短信验证码的发送次数为 1.

- 共享 session 信息

  - 在分布式系统下, 用户的请求会先打到负载均衡上, 然后再转发应用服务器上. 但负载均衡里有很多服务器, 它们并不保存用户的信息 (session), 也就不知道该将用户的请求转发到哪台应用服务器上. 此时, 如果有一个服务器专门存储 session 的话, 负载均衡就不需要关心将用户的请求转发到哪台应用服务器上的问题了, 负载均衡直接转发就行, 由应用服务器去向 session 服务器获取 session 信息, 而这个 session 服务器就是用 redis 实现的. 因为 session 信息本身就是一个个键值对, 而 redis 也是以键值对的形式组织数据的, 并且 redis 还速度非常快. 

    ![image-20231115212329979](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231115212329979.png)

## List

#### 内部编码

| 内部编码  | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| ziplist   | 当 value 个数较少或长度较小时, 使用该编码                    |
| linklist  | 当上面条件不满足时, 使用该编码                               |
| quicklist | Redis 3.2 引入, 替代了 ziplist 和 linklist. 它相当于 ziplist 和 linklist 的结合版 |

#### 应用场景

- 作为数组

- 作为消息队列

  - 利用 lpush 和 brpop 或者 rpush 和 blpop 可实现生产者消费者模型.

    ![image-20231115215316205](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231115215316205.png)

## Hash

#### 内部编码

| 内部编码  | 解释                                      |
| --------- | ----------------------------------------- |
| ziplist   | 当 value 个数较少或长度较小时, 使用该编码 |
| hashtable | 当上面条件不满足时, 使用该编码            |

#### 应用场景

- 缓存对象

  - 以用户信息为例, 它在关系型数据库 (mysql) 中的结构是这样的:

    ![image-20231117190336743](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231117190336743.png)

    它在 redis 中用 hash 存储的结构是这样的:

    ![image-20231117190325920](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231117190325920.png)

    - 该方式的空间代价较高, 需要控制 ziplist 和 hashtable 两种编码的转换

  - 用 string 类型 + json 格式也可以达到上述效果.

    - 该方式在序列化和反序列化上有一定的开销, 同时操作单个属性时, 非常不灵活

## Set

#### 内部编码

| 内部编码  | 解释                                        |
| --------- | ------------------------------------------- |
| intset    | 当 value 是整数, 并且个数较少时, 使用该编码 |
| hashtable | 当上面条件不满足时, 使用该编码              |

#### 应用场景

- 点赞
  - 由于 set 的特性, 所以可以保证每个用户只能对同一个 key 进行一次点赞
- 标签
  - set 能够存储该用户的一些特征, 而且在不同账号之间还可以利用 set 交集, 提取出公共标签
- 共同好友
- 统计 UV

## Zset

#### 内部编码

| 内部编码 | 解释                                      |
| -------- | ----------------------------------------- |
| ziplist  | 当 value 个数较少或长度较小时, 使用该编码 |
| skiplist | 当上面条件不满足时, 使用该编码            |

#### 应用场景

- 排行榜系统

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
