# MySQL 内外连接

## 内连接

内连接有两种方式：

-  `SELECT * FROM table1, table2 [, . . .] WHERE expr1, expr2 [, . . .]` 这种方式是最常见的内连接方式。
- `SELECT * FROM table1 INNER JOIN table2 ON expr1, expr2 [, . . .]` 这种方式是最规范的内连接方式。

**案例：**

显示 SMITH 的名字和部门名称：

- 常规内连接：

  ![image-20240406132025011](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240406132025011.png)

- 规范内连接：

  ![image-20240406132131282](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240406132131282.png)

## 外连接

外连接分为：

- 左外连接：`table1 LEFT [OUTER] JOIN table2 ON`，以 table1 为主，保证 table1 的行数不变。
- 右外连接：`table1 RIGHT [OUTER] JOIN table2 ON`，以 table2 为主，保证 table2 的行数不变。

**案例：**

准备表和数据：

![image-20240406132750125](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240406132750125.png)

左连接：查询所有学生的成绩，如果这个学生没有成绩也要显示出来这个学生的信息：

![image-20240406132956336](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240406132956336.png)

右连接：将所有的成绩都显示出来，即使没有学生与之对应：

![image-20240406133107156](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240406133107156.png)



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