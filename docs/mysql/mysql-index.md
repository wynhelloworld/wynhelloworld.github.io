# MySQL 索引

## 对索引的理解

一块磁盘，有很多机械设备，其中存储数据的设备叫做盘片，盘片的上下两面都能存储数据。盘面被划分为若干条同心圆（越靠近圆心半径越小），每一条同心圆都被叫做磁道。每条磁道都被平均的划分为若干条弧（每条磁道的弧的数量相等），每一条弧都被叫做扇区。而扇区就是磁盘存储数据的基本单位，每个扇区的大小默认为 512 字节（最经典的大小）。示意图如下：

![image-20240407141310062](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240407141310062.png)

操作系统与磁盘的数据交互基本单位是 4KB，也就是操作系统从磁盘中读取一次数据最少要选取 8 个扇区。

MySQL InnoDB 引擎与磁盘的数据交互基本单位是 16KB，也就是 MySQL 从磁盘中读取一次数据最少要选取 32 个扇区。（准确的讲，MySQL 是应用层软件，并不直接与磁盘交互，而是与操作系统交互，所以是 MySQL 与操作系统的数据交互基本单位是 16 KB。）这个基本单位在 MySQL 中被叫做 page。

当 MySQL 对某数据进行 CRUD 操作时，该数据一定是既在内存中，又在磁盘中，因为 MySQL 无法直接与磁盘数据进行交互，所以一定是操作系统将磁盘中的数据先读到内存，然后交给 MySQL，再由 MySQL 进行 CRUD 操作。为了提高效率，MySQL 会在启动时自动开辟一大块名为 Buffer Pool 的内存空间，将操作系统读取到的数据放在 Buffer Pool 中，然后进行 CRUD，操作完毕之后，再将 Buffer Pool 中的数据由操作系统刷新到磁盘中。而无论是从磁盘中读取还是刷新到磁盘中，数据交互的基本单位都是 page（MySQL 中的 page，一般是 16 KB）。

那么，Buffer Pool 中有很多的 page，如何进行管理？通过先描述再组织，一个 page 可以简单的认为由两个指针 prev 和 next 以及一块 数据 data 构成，每个 page 通过指针连接起来。只是简单的通过这两个指针连接起来，那么就是一条链表，而这个数据结构对于 MySQL 来说查询效率是非常差的。（数据查询速度的优化方式：数据结构和算法）所以 MySQL InnoDB 引擎使用 B+ 树来组织管理 page。

![image-20240407150817819](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240407150817819.png)

B+ 树的路上节点 page 仅有一个目录，而叶子节点 page 有两个指针还有数据。叶子节点 page 通过指针全部连接起来，叶子节点 page 内部的数据由链表组织，还添加了目录来提高查找效率。

MySQL InnoDB 就是以这样的方式来组织管理数据的。

那么，索引和上面 B+ 树结构有什么关系？一个索引就是一个 B+ 树结构，当给一个表中的某个列设置了索引时，B+ 树结构就会拿该列的值当作 dir 里的值。一个表可以给多个列设置多个索引，那么该表就会有多个 B+ 树结构（一般称为有多个索引）。但是只有主键索引的叶子节点的 page 里会存储数据，普通索引的叶子节点的 page 里不存储数据而存储的是该行的主键索引的值。

**聚簇索引和非聚簇索引**

- 聚簇索引：像 InnoDB 这样的 B+ 树的叶子节点中存储数据的索引叫做聚簇索引。
- 非聚簇索引：像 MyISAM 这样的 B+ 树的叶子节点中不存储数据，而是存储数据的地址的索引叫做非聚簇索引。

## 索引的操作

#### 创建索引

**创建主键索引**

- ```
  create table table_name (id int primary key, ...);
  ```

- ```
  create table table_name (id int, ..., primary key(id)); 
  ```

- ```
  create table table_name (id int, ...);
  alter table table_name add primary key(id);
  ```

- 当没有 primary key 关键字创建主键索引时，第一个 not null unique key 也可以用来创建主键索引

  ```
  create table table_name (id int not null unique key, ...);
  ```

**创建唯一键索引**

- ```
  create table table_name (id int unique, ...);
  ```

- ```
  create table table_name (id int, ..., unique(id)); 
  ```

- ```
  create table table_name (id int, ...);
  alter table table_name add primary(id);
  ```

**创建普通索引**

- ```
  create table table_name (id int, ..., index(id)); 
  ```

- ```
  create table table_name (id int, ...);
  alter table table_name add index(id);
  ```

- ```
  create table table_name (id int, ...);
  create index index_name on table_name(id);
  ```

#### 查询索引

- ```
  show keys from table_name;
  ```

- ```
  show index from table_name;
  ```

- ```
  desc table_name;
  ```

#### 删除索引

**删除主键索引**

```
alter table table_name drop primary key;
```

**删除其它索引**

- ```
  alter table table_name drop index index_name;
  ```

- ```
  drop index index_name on table_name;
  ```











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