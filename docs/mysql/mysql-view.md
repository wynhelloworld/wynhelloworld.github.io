# MySQL 视图

视图是一张表，由 select 的查询结果定义。视图与原表的数据是互通的，当通过视图修改数据时，会影响到原表的数据，当通过原表修改数据时，会影响到视图的数据。

定义如下：

```mysql
create view view_name as select . . .;
```

举例：

![image-20240412163106871](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163106871.png)

![image-20240412163125440](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163125440.png)

![image-20240412163200914](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163200914.png)

多了一张表结构，现在查看一下该表：

![image-20240412163245173](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163245173.png)

正是上面创建视图语句后面的 select 语句的查询结果

现在用视图来修改一下数据：

![image-20240412163559671](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163559671.png)

查看一下 dept 表：

![image-20240412163715429](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163715429.png)

可以发现通过视图修改数据，影响到了原表的数据

现在再来修改原表数据：

![image-20240412163842551](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163842551.png)

查看一下 vde 表：

![image-20240412163905784](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240412163905784.png)

可以发现，SMITH 变成了 zhangsan，也就是通过修改原表数据，影响到了视图数据

**删除视图**

```mysql
drop view view_name;
```

**视图的规则和限制**

- 与表一样，必须唯一命名(不能出现同名视图或表名)
- 创建视图数目无限制，但要考虑复杂查询创建为视图之后的性能影响
- 视图不能添加索引，也不能有关联的触发器或者默认值
- 视图可以提高安全性，必须具有足够的访问权限
- `order by` 可以用在视图中，但是如果从该视图检索数据 `select` 中也含有 `order by` 中的 `order by` 将被覆盖

- 视图可以和表一起使用



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

