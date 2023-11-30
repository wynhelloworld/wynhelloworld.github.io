# Redis 哨兵

在 Redis 主从复制模式下，如果 Master 发生故障而停止服务时，需要进行主从切换：从众多 Slave 中挑选出一个当作 Master（执行 slaveof no one 命令），并且要重新配置其它的 Slave 成为新的 Master 的 Slave（执行 slaveof master_ip master_port 命令）。

## 人工进行主从切换

若主从切换是人工进行的，那么通常需要搭配一个监控程序和一个报警程序。一但监控程序发现主节点发生故障而停止服务时，就让报警程序通过短信、电话、微信、邮件等等手段来通知程序员，然后程序员就赶快上线去修补故障，若能让主节点迅速恢复那就恢复，若不能迅速恢复就进行主从切换。但是这通常要花费很长时间（少则半个小时吧），而这是绝对不被容许的，所以 Redis 提出了**哨兵机制**去解决人工进行主从切换的问题。

## 哨兵机制自动进行主从切换

Redis Sentinel 是一个独立的进程。Sentinel 节点会监控所有的 Master、Slave 和其余 Sentinel，一但发现有节点出故障了，就令该节点下线。若某个 Sentinel 发现 Master 出故障了，仅凭该 Sentinel 无法下定论让 Master 下线（因为该 Sentinel 可能误判），所以该 Sentinel 会与其它的 Sentinel 进行商讨，若达成共识认为 Master 出故障了，那么这些 Sentinel 会从内部挑选一个 Leader，让 Leader 去执行主从切换的工作。

#### 使用 Docker 搭建 3 Sentinel 1Master 2 Slave 架构

Sentinel 的个数一般是奇数个，为了方便选举 Leader。

![image-20231130215540198](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231130215540198.png)





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