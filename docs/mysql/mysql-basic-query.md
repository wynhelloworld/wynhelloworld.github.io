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

> 全列插入时，table_name 后面的 (column, …) 可以省略。不是全列插入时，不可以省略。

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







## Update



## Delete







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