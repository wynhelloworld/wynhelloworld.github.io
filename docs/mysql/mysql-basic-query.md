# MySQL 基础查询

针对表中的数据可进行以下四种操作（CRUD）：

- Create（创建）
- Retrieve（读取）
- Update（更新）
- Delete（删除）

## Create

**语法：**

```
INSERT [INTO] table_name [(column, ...)] VALUES (value_list) [, (value_list), ...]
```

> 全列插入时，table_name 后面的 (column, . . .) 可以省略。不是全列插入时，不可以省略。

先创建一个表：

![image-20231226193825001](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226193825001.png)

单行全列插入：

![image-20231226194054971](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226194054971.png)

多行指定列插入：

![image-20231226194259389](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226194259389.png)

由于主键或者唯一键对应的值以及存在而导致插入失败时，可以选择性的进行同步更新操作：

语法：

```
INSERT INTO ... ON DUPLICATE KEY UPDATE column=value [, column=value, ...]
```

> `ON DUPLICATE KEY` ：指当发生重复 key 的时候。

- 失败情况：

  ![image-20231226194549956](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226194549956.png)

- 失败时进行同步更新：

  ![image-20231226194750594](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226194750594.png)

  > - `Query OK, 0 rows affected`：有冲突，并且 update 后面的值一样，表中数据无变化。
  > - `Query OK, 1 rows affected`：无冲突，要插入的值直接插进去了，表中数据多了一行。
  > - `Query OK, 2 rows affected`：有冲突，要插入的值被无视，表中的数据根据 update 后面的值进行了修改。

当主键冲突时，还可以直接把表中的该条数据删掉，插入新的数据，用 `REPLACE` 语句。

![image-20231226200858979](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226200858979.png)

> - `Query OK, 1 rows affected`：无主键 / 唯一键冲突，直接插入数据。
>
> - `Query OK, 2 rows affected`：有主键 / 唯一键冲突，原数据先被删除，后插入新数据。

## Retrieve

**语法：**

```
SELECT 
    [DISTINCT] {* | column [, column] ...}
    FROM table_name
    [WHERE ...]
    [ORDER BY column [ASC | DESC], ...]
    LIMIT ...
```

#### Select 语句

创建表结构：

![image-20231226203017645](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226203017645.png)

插入测试数据：

![image-20231226203631742](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226203631742.png)

全列查询：

![image-20231226203752426](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226203752426.png)

指定列查询：

![image-20231226203834293](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226203834293.png)

查询字段为表达式：

![image-20231226204013496](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226204013496.png)

为查询结果添加别名：

```c++
SELECT column [AS] alias_name FROM table_name
```

![image-20231226204333549](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226204333549.png)

结果去重：

![image-20231226204534396](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231226204534396.png)

> 原来有 2 个 98 分，现在就只有 1 个 98 分。

#### Where 条件

Where 后面的都是条件，条件由列字段配合比较运算符和逻辑运算符构成。

**比较运算符：**

| 运算符             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| >, >=, <, <=       | 大于，大于等于，小于，小于等于                               |
| =                  | 等于，NULL 不安全，例如 NULL = NULL 的结果是 NULL            |
| <=>                | 等于，NULL 安全，例如 NULL = NULL 的结果是 TRUE(1)           |
| !=, <>             | 不等于                                                       |
| BETWEEN a0 AND a1  | 返回 value 属于 [a0, a1] 中的所有行                          |
| IN (option, . . .) | 若 value 是 option 中的某一个，则返回该行                    |
| IS NULL            | 若 value 是 NULL，则返回该行                                 |
| IS NOT NULL        | 若 value 不是 NULL，则返回该行                               |
| LIKE               | 模糊匹配。%表示任意多个（包括 0 个）字符，_ 表示任意一个字符 |

**逻辑运算符：**

| 运算符 | 说明                                              |
| ------ | ------------------------------------------------- |
| AND    | AND 两边的条件都为 TRUE，总的结果才为 TRUE        |
| OR     | OR 两边的条件有任意一个为 TRUE，总的结果就为 TRUE |
| NOT    | NOT 后边的条件为 FASLE，总的结果才为 TRUE         |

查询数学成绩 >= 80 分以上的同学：

![image-20240402195727564](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240402195727564.png)

NULL 的查询：

![image-20240402195955878](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240402195955878.png)

#### 结果排序

**语法：**

```
SELECT ... FROM table_name [WHERE ...] ORDER BY column [ASC | DESC], [...];

	ASC 为升序
	DESC 为降序
	默认为 ASC
```

> 没有 ORDER BY column [ASC | DESC] 子句返回的结果的顺序是为定义的！

查询数学成绩 >= 80 分以上的同学，按升序显示结果：

![image-20240402200735556](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240402200735556.png)

#### 筛选分页结果

**语法：**

```
从偏移量 0 开始，该页有 n 行
SELECT ... FROM table_name [WHERE ...] [ORDER BY ...] LIMIT n;
从偏移量 s 开始，该页有 n 行
SELECT ... FROM table_name [WHERE ...] [ORDER BY ...] LIMIT s, n;
从偏移量 s 开始，该页有 n 行
SELECT ... FROM table_name [WHERE ...] [ORDER BY ...] LIMIT n OFFSET s;
```

> 对未知表进行查询时，最好加上 LIMIT n，n 为 个位数，避免因为表中数据过大，查询全表数据导致数据库卡死。

按数学降序成绩进行分页，每页 3 条记录：

![image-20240402203702309](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240402203702309.png)

## Update

**语法：**

```
UPDATE table_name SET column = expr [, column = expr ...] [WHERE ...] [ORDER BY ...] [LIMIT ...]
```

修改孙权的数学成绩为 90 分：

![image-20240403150504716](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403150504716.png)

修改所有同学的语文成绩为 60 分：

![image-20240403150643298](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403150643298.png)

## Delete

**格式：**

```
DELETE FROM table_name [WHERE ...] [ORDER BY ...] [LIMIT ...]
```

删除宋公明的成绩：

![image-20240403151533121](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403151533121.png)

删除整张表的数据，先准备一个测试表：

![image-20240403152010470](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403152010470.png)

然后插入数据：

![image-20240403152119905](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403152119905.png)

删除整张表的数据：

![image-20240403152210125](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403152210125.png)

查看表结构：

![image-20240403152320748](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403152320748.png)

能够看到有一项为 AUTO_INCREMENT=4，则意味着，使用 DELETE 删除整张表数据的时候，不会重置 AUTO_INCREMENT 项。

**截断表：**

```
TRUNCATE [TABLE] table_name
```

> 1. 只能对整张表使用，不能对表中的某几个条目进行操作；
> 2. 实际上，MySQL 不对数据进行操作，所以比 DELETE 速度要快；
> 3. BRUNCATE 操作不经过事务，所以不支持回滚；
> 4. 会重置 AUTO_INCREMENT 项。

准备测试表和数据：

![image-20240403152946043](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403152946043.png)

进行截断：

![image-20240403153016188](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403153016188.png)

可以看到，影响了其中的 0 rows，所以实际并没有对数据进行操作。

查看表结构：

![image-20240403153121508](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403153121508.png)

没有 AUTO_INCREMENT 这一项，所以重置了该项。

#### 插入查询结果

用到的语法：

```
创建 table1 使得结构与 table2 一模一样：
	CREATE TABLE table1 LIKE table2;
将 table2 的查询出来的结果插入 table1 中：
	INSERT INTO table1 SELECT * FROM table2;
```

删除表中的重复数据，先准备测试表和测试数据：

![image-20240403154021252](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403154021252.png)

准备备份表：

![image-20240403154204903](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403154204903.png)

将不重复的查询结果插入到备份表中：

![image-20240403154317633](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403154317633.png)

修改表名字：

![image-20240403154449662](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403154449662.png)

查询结果，实现表的去重操作：

![image-20240403154525823](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403154525823.png)

## 聚合函数

| 函数                   | 说明                                       |
| ---------------------- | ------------------------------------------ |
| COUNT([DISTINCT] expr) | 返回查询到的数据的总量                     |
| SUM([DISTINCT] expr)   | 返回查询到的数据的总和，不是数字没有意义   |
| AVG([DISTINCT] expr)   | 返回查询到的数据的平均值，不是数字没有意义 |
| MAX([DISTINCT] expr)   | 返回查询到的数据的最大值，不是数字没有意义 |
| MIN([DISTINCT] expr)   | 返回查询到的数据的最小值，不是数字没有意义 |

统计表中的行数：

![image-20240403160323556](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403160323556.png)

统计表中的数学成绩的总分：

![image-20240403160448774](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403160448774.png)

统计表中的数学成绩的平均分、最高分和最低分：

![image-20240403160558268](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403160558268.png)

## group by 子句的使用

在 select 语句中使用 group by 可以达到按照指定列进行分组（分表）的效果。

语法：

```
SELECT ... FROM table_name GROUP BY column;
	SELECT 后面的 ... 一般跟的是 GROUP BY 后面的指定列
```

准备测试表和测试数据，这里使用 oracle 9i 的经典测试表：

- EMP 员工表
- DEPT 部门表
- SALGRADE 工资等级表

显示每个部门的平均工资和最高工资：

![image-20240403170223019](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403170223019.png)

显示每个部门的每种岗位的平均工资和最低工资：

![image-20240403170725358](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403170725358.png)

显示平均工资低于 2000 的部门和它的平均工资：

![image-20240403170903187](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403170903187.png)

> having 子句的应用场景是用于分组聚合统计之后的条件筛选。和 where 的作用类似但应用场景不同。



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