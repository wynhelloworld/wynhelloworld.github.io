---
title: Redis常见数据类型及使用
date: 2023-11-08 16:53:35
tags: Redis
categories: Redis
index_img: /img/redis-icon.png
excerpt: 对Redis常见数据类型及其相关命令的总结
---

# 基本全局命令

Redis有5种常见数据类型, 但它们都是值的类型, 而键的类型只有字符串类型. 所以, 对于键, 有一些通用的命令.

| Commands                                  | Time complexity | Effect                       |
| ----------------------------------------- | --------------- | ---------------------------- |
| KEYS pattern                              | O(N)            | 返回所有匹配pattern的key     |
| EXISTS key [key ...]                      | O(N)            | 判断key是否存在              |
| DEL key [key ...]                         | O(N)            | 删除key                      |
| EXPIRE key seconds [NX \| XX \| GT \| LT] | O(1)            | 给key设置过期时间            |
| TTL key                                   | O(1)            | 返回key剩余的过期时间        |
| TYPE key                                  | O(1)            | 返回key对应的value的类型     |
| OBJECT ENCODING key                       | O(1)            | 返回key对应的value的编码方式 |

**注意事项:**

* `KEYS`命令的时间复杂度为O(N), 这里的N指的是数据库中key的数量, 所以`KEYS *`命令慎用或者不用, 因为极有可能使redis服务器被阻塞, 导致redis服务器无法向其它客户端提供服务. 

  redis经常被用作缓存, 替mysql负重前行, 而如果redis被阻塞了, 那么就会有大量请求打到mysql上, 导致mysql瘫痪.

*  `EXISTS`和`DEL`的时间复杂度都是O(N), 这里的N指的是命令后跟着的key的数量, 所以, 这里的O(N)也可以看作O(1). 

  为什么有些命令能够同时操作多个key呢? 

  这是因为, redis是一个C/S设计程序, 每一次命令的执行都意味着一次网络通信, 所以为了提高效率, redis的很多命令都支持一次操作多个key.

* redis的key的过期策略是如何实现的?

  redis的整体策略是: 定期删除、惰性删除、一系列内存淘汰策略.

  * 定期删除: 过一段时间, 抽取一部分key, 验证过期时间 (抽取验证的过程要非常快, 为了防止redis被阻塞);
  * 惰性删除: key到达过期时间了, 但此时不删除它, 而是等下一次访问它时再进行删除.

* redis的单线程指的是: 所有的命令请求是由一个线程完成的! 上面所说的redis被阻塞, 也是因为这个原因.

# String类型

| Commands                     | Time complexity | Effect                               |
| ---------------------------- | --------------- | ------------------------------------ |
| SET key value                | O(1)            | 创建键值对                           |
| GET key                      | O(1)            | 返回key对应的value                   |
| MSET key value [key value …] | O(N)            | 创建多个键值对                       |
| MGET key [key …]             | O(N)            | 返回多个key对应的value               |
| SETNX key value              | O(1)            | 当key未曾存在过时, 创建键值对        |
| SETEX key seconds value      | O(1)            | 创建键值对, 并且设置过期时间(单位s)  |
| PSETEX                       | O(1)            | 创建键值对, 并且设置过期时间(单位ms) |
| INCR                         |                 |                                      |
| INCRBY                       |                 |                                      |
| DECR                         |                 |                                      |
| DECRBY                       |                 |                                      |
| INCRBYFLOAT                  |                 |                                      |
| APPEND                       |                 |                                      |
| GETRANGE                     |                 |                                      |
| SETRANGE                     |                 |                                      |
| STRLEN                       |                 |                                      |
|                              |                 |                                      |

