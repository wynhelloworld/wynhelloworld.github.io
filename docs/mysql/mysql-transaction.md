# MySQL 事务

## 对事务的理解

举个例子，比如一个人去银行存钱，客户只需要把钱和相关信息提供给工作人员就行了，然后工作人员需要做很多操作（接收清点现金、记录存款信息、更新账户信息、. . .），然后告诉客户存钱是否成功。对于客户来说，客户提供钱和相关信息，工作人员给客户存钱是否成功的结果，这件事就是一个事务，是原子的。对于工作人员来说，这个事务需要做很多的操作，如果中间有一步出现任何问题了，那么就要回滚到做这个事务之前的状态。

所以，在数据库中，事务就是一组 DML 语句，这些语句要共同完成一个目标，这些语句要么全部成功，要么全部失败，是一个整体，这个整体是原子性的。在数据库中，是有很多的事务会同时运行的，每个事务有多个 DML 语句组成，这些语句是可能会同时访问同一张表的同一条记录的，这时就会存在并发问题。所以一个完整的事务绝对不仅仅是几条 SQL 语句的组合，同时还要满足以下性质（**ACID**）：

- 原子性（**A**tomicity）
- 一致性（**C**onsistency）
- 隔离性（**I**solation）
- 持久性（**D**urability）

## 对事务的操纵

#### 查看支持事务的引擎

```mysql
show engines;   -- 表格显示	
show engines\G  -- 行显示
```

> MySQL 中仅 InnoDB 支持事务。

#### 事务的提交方式

**自动提交**

自动提交是默认开启的，可通过以下命令查看：

```mysql
show variables like 'autocommit';
```

可通过以下命令来开启和关闭自动提交：

```mysql
set [global] autocommit=0;	// 关闭事务自动提交
set [global] autocommit=1;	// 开启事务自动提交
```

在自动提交开启的情况下，每一条 DML 语句都是一个事务。

**手动提交**

手动提交和自动提交是互不影响的，在自动提交开启的情况下，也可以进行手动提交。

事务的手动提交有相关的以下几个指令：

- `start transaction` or `begin` ：开启事务
- `savepoint name` ：设置保存点，保存点名字为 name
- `rollback [to name]` ：回滚到保存点 name，若不添加 name，则回滚到事务开启前
- `commit` ：提交事务（关闭事务）

**演示手动提交**

```mysql
mysql> create table stu (
    -> id int primary key auto_increment,
    -> name varchar(20),
    -> gender varchar(20)
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into stu(name, gender) values('wyn', 'male');
Query OK, 1 row affected (0.00 sec)

mysql> savepoint p1;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into stu(name, gender) values('cl', 'female');
Query OK, 1 row affected (0.00 sec)

mysql> savepoint p2;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from stu;
+----+------+--------+
| id | name | gender |
+----+------+--------+
|  3 | wyn  | male   |
|  4 | cl   | female |
+----+------+--------+
2 rows in set (0.00 sec)

mysql> rollback to p1;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from stu;
+----+------+--------+
| id | name | gender |
+----+------+--------+
|  3 | wyn  | male   |
+----+------+--------+
1 row in set (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from stu;
Empty set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)

```

自动提交是演示不出来的，因为一条 DML 语句就是一个事务，可以把自动提交想象为如下：

```mysql
begin;
DML 语句
commit;
```

## 事务的隔离级别

MySQL 服务端是会被大量客户端同时以事务的形式访问的，每一个事务都可能包含多条 SQL，虽然说事务对用户表现出来的特性是原子的，但是每个事务（多条 SQL）执行总需要时间，而在这段时间内，就可能会存在不同事务中的 SQL 访问同一张表的同一条记录的情况，而为了保证事务在执行时尽量不受干扰就产生了隔离性，而允许不同事务之间受到不同程度的干扰就产生了隔离级别：

- 读未提交（Read Uncommitted）：在该隔离级别下，一个事务在它的执行过程中是可以看到其它事务未提交的执行结果的，相当于没有隔离性。实际生产中不会采用该隔离级别，因为会出现很多并发问题，如脏读、不可重复读、幻读等

- 读已提交（Read Committed）：在该隔离级别下，一个事务在它的执行过程中是可以看到其它事务提交之后的执行结果的，也是大多数数据库的默认隔离级别（非 MySQL）。该隔离级别解决了脏读，但并未解决不可重复读和幻读
- 可重复读（Repeatable Read）：是 MySQL 的默认隔离级别。在该隔离级别下，一个事务在它的执行过程中所看到的数据是完全一致的，解决了脏读和不可重复读，但并未解决幻读（MySQL 解决了幻读）
- 串行化（Serializable）：最高隔离级别，强制事务进行排序，每条记录在进行读读时加共享锁，读写和写写时加排它锁，这种隔离级别太极端，容易导致锁竞争和超时

#### 隔离级别的操作

**查看隔离级别**

```mysql
select @@global.tx_isolation;   -- 查看全局隔离级别
select @@session.tx_isolation;  -- 查看会话（当前）隔离级别
select @@tx_isolation;          -- 查看会话（当前）隔离级别
```

**设置隔离级别**

```mysql
set {session | global} transaction isolation level {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ	| SERIALIZABLE}
```

**读未提交——效果演示**

设置会话 A、B 都为 READ UNCOMMITTED 隔离级别：

```mysql
set session transaction isolation level READ UNCOMMITTED;
```

现有如下数据：

```mysql
mysql> select * from stu;
+------+------+--------+
| id   | name | gender |
+------+------+--------+
|   29 | wyn  | male   |
+------+------+--------+
1 row in set (0.00 sec)
```

按表格给出的顺序执行：

![image-20240411203908131](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240411203908131.png)

也就是说，两个事务并发执行过程中，事务 B 看到了事务 A 还未 commit 时的结果。

**读已提交——效果演示**

设置会话 A、B 都为 READ UNCOMMITTED 隔离级别：

```mysql
set session transaction isolation level READ COMMITTED;
```

现有如下数据：

```mysql
mysql> select * from stu;
+------+------+--------+
| id   | name | gender |
+------+------+--------+
|   29 | wyn  | male   |
+------+------+--------+
1 row in set (0.00 sec)
```

按表格给出的顺序执行：

![image-20240411204409831](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240411204409831.png)

从结果可以看出，在两个事务并发执行过程中，事务 B 看不到事务 A 还未 commit 的结果，但可以看到事务 A commit 之后的结果。

**可重复读——效果演示**

设置会话 A、B 都为 READ UNCOMMITTED 隔离级别：

```mysql
set session transaction isolation level REPEATABLE READ;
```

现有如下数据：

```mysql
mysql> select * from stu;
+------+------+--------+
| id   | name | gender |
+------+------+--------+
|   29 | wyn  | male   |
+------+------+--------+
1 row in set (0.00 sec)
```

按表格给出的顺序执行：

![image-20240411205641319](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240411205641319.png)

从结果可以看出，事务 B 永远都是一样的，只有在事务 B commit 之后，才能看到最新数据。

**串行化——效果演示**

设置会话 A、B 都为 READ UNCOMMITTED 隔离级别：

```mysql
set session transaction isolation level SERIALIZABLE;
```

现有如下数据：

```mysql
mysql> select * from stu;
+------+------+--------+
| id   | name | gender |
+------+------+--------+
|   29 | wyn  | male   |
+------+------+--------+
1 row in set (0.00 sec)
```

按表格给出的顺序执行：

![image-20240411211050934](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240411211050934.png)

在这种隔离模式下，事务之间是严格按照顺序执行的，读读之间采用共享锁（读锁），读写和写写之间采用排它锁（写锁）。

**隔离级别与影响**

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 加锁读 |
| -------- | ---- | ---------- | :--- | ------ |
| 读未提交 | YES  | YES        | YES  | 不加锁 |
| 读以提交 | NO   | YES        | YES  | 不加锁 |
| 可重复读 | NO   | NO         | NO   | 不加锁 |
| 串行化   | NO   | NO         | NO   | 加锁   |

> 可重复读，MySQL InnoDB 解决了幻读问题，其它数据库是可能出现幻读问题的。

- 脏读：读取到了别的事务还未 commit 的结果
- 不可重复读：在同一个事务中多次读取相同记录，取得了不同的结果
- 幻读：是不可重复读的一种，在同一个事务中，比如用 `select * from table_name;`  第一次读取还未出现的数据，第二次读取就出现了，这是因为其它事务 `insert` 了新的数据

## MVCC 机制

MySQL 的并发场景无非就是三种：

- 读读：不存在任何安全问题，不需要并发控制
- 读写：有线程安全问题，可能会出现脏读、不可重复读、幻读
- 写写：有线程安全问题，可能会出现数据更新丢失问题

对于后两种会出现线程安全问题的并发场景来说，出现次数最多的就是读写场景了，而对于读写场景若是还加锁的话，那么就会极大的降低事务的并发效率，于是人们就提出了一种基于乐观锁思想的多版本并发控制（Multi-Version Concurrency Control）机制。

MVCC 能够为数据库解决如下问题：

- 在读写并发场景下，不需要加锁，从而提高事务的并发效率
- 可以解决脏读、不可重复读、幻读问题，但没有解决数据更新丢失问题

要了解 MVCC 机制，需要依次了解以下知识：

- 3 个记录隐藏字段
- undo log
- Read View

**3 个列隐藏字段**

- `DB_TRX_ID`：6 byte，创建该条记录/最后一次修改该条记录的事务 ID
- `DB_ROLL_PTR`：7 byte，指向该条记录的上一个版本
- `DB_ROW_ID`：6 byte，隐藏的自增 ID（隐藏主键），当表中没有显式指定主键时，MySQL 会以该字段来生成聚簇索引
- 其实还有一个隐藏的字段 flag，用来标记该记录是否被删除

**undo log**

undo log 会将记录的各个版本记录下来，以供回滚操作和支持 MVCC 机制。这里将 undo log 简单理解为 MySQL Buffer Pool 中的一段内存缓冲区即可。

undo log 配合上面几个字段就能够实现一条记录的多版本链，如下：

假设事务 10 首次创建了一条记录：

![image-20240412154009260](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412154009260.png)

由于该记录是首次创建，所以并没有历史版本，所以回滚指针指向 NULL，undo log 中也没有该记录的历史版本。

这时事务 11 要修改该条记录，那么此时会将最新记录拷贝一份到 undo log 中，然后修改最新记录：

![image-20240412153748515](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412153748515.png)

之后事务 12 又要修改该条记录：

![image-20240412153837624](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412153837624.png)

于是就这样一步一步，在 undo log 中就有了该条记录的所有历史版本。

> undo log 里的所有记录，都肯定是之前已经 commit 的结果
>
> 而最新记录不一定是 commit 的结果

我们经常把 undo log 里的记录都叫做快照。

当前读：所有的针对最新记录的操作，我们都叫做当前读，比如 update、delete（不会删除数据，而是会将 flag 置为 false），以及 select 最新版本（这时需要加共享锁，也就是串行化了）

快照读：select 历史版本就叫做快照读（普通的 select 都是快照读）

**Read View**

在 RC 和 RR 模式下，进行普通 select 时，会形成一个 Read View，它的结构如下：

```c++
class ReadView { // 省略...
private:
/** 高水位，大于等于这个ID的事务均不可见 */ 
  trx_id_t m_low_limit_id
 
/** 低水位:小于这个ID的事务均可见 */ 
  trx_id_t m_up_limit_id;
  
/** 创建该 Read View 的事务ID */ 
  trx_id_t m_creator_trx_id;
  
/** 创建视图时的活跃事务id列表 */ 
  ids_t m_ids;
  
/** 配合purge，标识该视图不需要小于m_low_limit_no的UNDO LOG，
* 如果其他视图也不需要，则可以删除小于m_low_limit_no的UNDO LOG*/
  trx_id_t m_low_limit_no; 
  
/** 标记视图是否被关闭 */
  bool m_closed;
// 省略... 
};
```

```c++
m_ids;          // 用来记录 Read View 生成时刻，系统正活跃的所有事务 ID
up_limit_id;    // 用来记录 m_ids 中最小的事务 ID
low_limit_id;   // 用来记录 m_ids 中最大的事务 ID 值 +1
creator_trx_id  // 用来记录创建该 Read View 的事务 ID
```

那么以上四个变量的作用是什么？

配合 undo log 里的版本链，再配合以下算法从而决定当前读能读到哪条记录：

![image-20240412160351060](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412160351060.png)

当进行快照读时，会从最新记录开始，拿最新记录的 DB_TRX_ID 来分别进行上述 4 次比较，若返回 true，则意味着当前进行快照读的事务能看见该条最新记录，若返回 false，则看不到最新记录，于是就拿着最新记录的 DB_ROLL_PTR 找到 undo log 里的记录，重复上述操作，总能找到一条能够看到的记录。

根据上述算法，可以得出结论：当 select 形成 Read View 时，在该时刻之前 commit 的记录，select 都能看到，在该时刻之后 commit 的记录，select 都看不到。

####  RC 和 RR 的区别是什么？

RC 模式下，快照读能读取到一条记录多次 commit 的结果

RR 模式下，快照读仅能读取到一条记录一次 commit 的结果

所以区别就是，RC 模式下，每次进行快照读都会生成 Read View，RR 模式下，每次进行快照读都只会在第一次时生成 Read View。



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