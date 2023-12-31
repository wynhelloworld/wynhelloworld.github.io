# Redis 事务

Redis 事务指的是将一组命令打包起来，让 Redis 服务器依次执行这组命令，在执行这组命令的过程中，不会被其它的命令插队。

## Redis 事务与 ACID

- 原子性（Atomicity）：Redis 事务中无原子性（或者是有弱化版的原子性）。因为原子性本来的概念是指**要么全部做完，要么全部不做**，而 MySQL 中的原子性是指**要么全部做完且成功，要么全部不做**。正因为 MySQL 提高了原子性的门槛，所以才说 Redis 事务无原子性（或者是有弱化版的原子性）。Redis 官方本来有句话是说 Redis 事务是有原子性的，但后来 Redis 官方删除了这句话。
- 一致性（Consistency）：Redis 事务中无一致性。Redis 不支持回滚，也没有一些校验规则，所以 Redis 并不保证事务执行前后的内容一致。
- 隔离性（Isolation）：Redis 事务中无隔离性。Redis 使用单线程来处理所有的命令，不存在并发情况。
- 持久性（Durability）：Redis 事务中无持久性。Redis 是一个内存数据库，虽然提供了持久化机制，但是 Redis 的目的并不是为了数据的持久，Redis 持久化机制与事务的持久性没有任何关系。
## Redis 事务的操作

Redis 事务本质上是在服务器上搞了个**事务队列**（每个客户端对应一个）。当客户端开启事务后，所有的命令会发送到服务器的事务队列中，而不是服务器就直接运行命令了，当客户端发送执行事务的命令时，服务器才去执行事务队列中的命令。

Redis 事务的相关操作命令如下：

- `Multi`：开启事务。开启事务之后，客户端发送的所有命令都会跑到服务器的事务队列中，等待执行。
- `Exec`：执行事务。让服务器执行事务队列中的命令。
- `Discard`：取消事务。让服务器清空并关闭事务队列。
- `Watch`：监控 Key。在开启事务之前调用，在执行事务之时作用。被监控的 Key 若是被事务之外的命令所修改了，那么整个事务就都不执行了。
- `UnWatch`：取消监控 Key。

## Watch 的原理

Watch 很像乐观锁，被监控的 Key 会拥有一个版本号，当 Key 发生修改时，版本号都会变化（变大）。在事务中修改 Key 时，也会使得版本号发生变化，并且还会记录下该版本号，当执行 Exec 时，会检查 Key 当前版本号与之前记录的版本号是否相同，若相同则执行事务，反之则丢弃事务。



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
