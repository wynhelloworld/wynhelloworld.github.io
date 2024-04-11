# MySQL 事务

## 对事务的理解

举个例子，比如一个人去银行存钱，客户只需要把钱和相关信息提供给工作人员就行了，然后工作人员需要做很多操作（接收清点现金、记录存款信息、更新账户信息、. . .），然后告诉客户存钱是否成功。对于客户来说，客户提供钱和相关信息，工作人员给客户存钱是否成功的结果，这件事就是一个事务，是原子的。对于工作人员来说，这个事务需要做很多的操作，如果中间有一步出现任何问题了，那么就要回滚到做这个事务之前的状态。

所以，在数据库中，事务就是一组 DML 语句，这些语句要共同完成一个目标，这些语句要么全部成功，要么全部失败，是一个整体，这个整体是原子性的。在数据库中，是有很多的事务会同时运行的，每个事务有多个 DML 语句组成，这些语句是可能会同时访问同一张表的同一条记录的，这时就会存在并发问题。所以一个完整的事务绝对不仅仅是几条 SQL 语句的组合，同时还要满足以下性质（**ACID**）：

- 原子性（**A**tomicity）
- 一致性（**C**onsistency）
- 隔离性（**I**solation）
- 持久性（**I**solation）

## 对事务的操纵

#### 查看支持事务的引擎

```mysql
show engines;		-- 表格显示	
show engines\G	-- 行显示
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
select @@global.tx_isolation;		-- 查看全局隔离级别
select @@session.tx_isolation;	-- 查看会话（当前）隔离级别
select @@tx_isolation;					-- 查看会话（当前）隔离级别
```

**设置隔离级别**

```mysql
set {session | global} transaction isolation level {READ UNCOMMITTED |
																										READ COMMITTED   |
																										REPEATABLE READ	 |
																										SERIALIZABLE}
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