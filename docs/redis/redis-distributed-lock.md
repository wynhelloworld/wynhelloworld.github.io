# Redis 分布式锁

## 为什么需要分布式锁？

为了保护临界资源的安全，才诞生了锁。许多库都提供了锁机制，比如 C++ 的 STL 库中的 mutex，但这些都是基于多线程的，也就是作用在单进程内部的。

在分布式这种多进程多主机的场景下，上述库中给予的锁的机制就无能为力了。

## Redis 分布式锁如何实现？

用一台 Redis 服务器专门存储锁（其实就是键值对），当用户去访问临界资源时，先去「锁服务器」上加锁（用 setnx 设置键值对），若成功就能访问临界资源，若不能成功则意味着已经有人先加了锁，先访问临界资源了。

![image-20231213212655169](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231213212655169.png)

但是，单纯的使用 setnx 去「锁服务器」上加锁，很有可能出现死锁：

- 业务逻辑出现问题，没有及时使用 del 释放锁。
- 客户端宕机了。

那么如何解决上述问题？

## 如何避免死锁？

只需要在 setnx 之后，用 expire 加上过期时间即可。但这个方法不可行，因为这是两条命令，这样加锁并不是原子的，非常容易出现各种问题。所以正确做法应该是使用 set nx ex 加锁。

但是，还有问题，锁服务器是任何客户端都能访问的，上面的锁任何客户端都能看到并且能够释放，那么假设 B 将 A 设置的锁给释放了，然后再自己加锁，这不乱套了吗？

那么如何解决上述问题？

## 如何避免别人释放自己的锁？

其实，上面的场景是不太合适的，因为不会有人在写自己的产品时，专门去「搞破坏」，但是总会出现别人「误删」了不属于自己的锁的情况。

锁其实就是键值对中的 key，那么我们在设置键值对的时候将 value 设置成自己的标识符，比如 uuid、runid等。当释放锁的时候，先进行校验，根据 value 判断该锁是否属于自己，属于自己的话再进行释放。

那么执行这一系列操作，肯定需要很多步骤，那么如何保证原子性 ？这时就要引入 lua 脚本了。

代码如下：

```lua
if redis.call('get',KEYS[1]) == ARGV[1] then
    return redis.call('del',KEYS[1])
else
    return 0
end;
```

将这份 .lua 文件发送给「锁服务器」，由于 Redis 处理请求是单线程的，所以它在处理这个 lua 脚本时，并不会被其它的请求干扰。

锁被别人误删的问题解决了，但锁设置了过期时间，过期时间到了也会自动被删除，若把握不好过期时间的长短，也容易出现问题：

- 过期时间太短，临界资源还没看完，锁就释放了，导致别人也进来看临界资源了。
- 过期时间太长，若先前占据锁的客户端宕机了，锁还没有及时被释放，导致其它客户端这期间进不来。

那么如何解决上述问题？

## 如何评估锁的过期时间？



## Redlock 算法





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