# MySQL 表的约束

## 空属性

通过 `desc` 命令查看表的结构的时候，会出现一个 `Null` 字段，该字段默认是 Yes ，也就是默认允许该列值不设置数据的时候为 NULL。

![image-20231222182726583](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222182726583.png)

而在实际场景中，数据为空是不好的做法，所以为了限制程序员必须给该列设置数据，就设置该列的 `Null` 字段为 No。

通过在设置字段的时候加上 `not null` 即可。

![image-20231222182422118](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222182422118.png)

## 默认值 

通过 `desc` 命令查看表结构的时候，还会出现一个 `Default` 字段，该字段默认是 NULL，也就是当该列值不设置值时默认值就为 NULL。

通过上面的图也能看到该字段。

在建表设置字段的时候，可以通过 `default xxx` 指定默认值，示例如下：

![image-20231222183423595](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222183423595.png)

## 列描述

当一张表中的字段很多时，就容易忘记某个字段创建时的含义。所以可以在建表的时候给字段加上描述，通过 `comment 'xxx'`。

通过 `SHOW CREATE TABLE table_name\G;` 命令查看。

![image-20231222184248878](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222184248878.png)

## zerofill

设置整数的时候，可以在后面加个括号，填上整数。示例如下：

![image-20231222184548584](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222184548584.png)

该数字的作用只有在设置 `zerofill` 约束的时候才体现，作用就是当查看数据的时候最少显示 6 位，不足 6 位的整数高位补 0 。

可以在建表的时候给字段设置上 `zerofill` 属性。

![image-20231222185152046](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222185152046.png)

> 只是显示的时候高位补了 0，实际存储的还是 2023，而不是 002023。

## 主键 

主键用来约束某个字段不能为 NULL，不能重复，一张表中最多只能有一个主键，且主键通常是整数。

有以下几种方式设置主键：

- 在字段后面写上 `primary key`：

  ![image-20231222190141117](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222190141117.png)

- 字段写完之后，在全部字段的后面写上 `primary key(xx)`，xx 就是要设置的主键的字段：

  ![image-20231222190158769](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222190158769.png)

- 还可以使用 `ALTER TABLE table_name ADD PRIMARY KEY(xx)`命令设置主键，xx 就是要设置的主键的字段：

  ![image-20231222190507627](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222190507627.png)

- 还可以给表新增一个字段，然后设置该字段为主键：

  ![image-20231222190832470](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222190832470.png)

> - 设置主键时，只能设置一个；
> - 表中已经有主键时，就不能再设置主键了。

## 复合主键

复合主键和主键作用差不多。

有以下几种方式设置复合主键：

- 建表的时候，在所有的字段的后面写上 `primary key(xx, xx, ...)`，括号里就是要添加的复合主键：

  ![image-20231222191337770](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222191337770.png)

- 在建完表之后，可以通过 `ALTER TABLE table_name ADD PRIMARY KEY(xx, xx, ...)` 命令来设置复合主键，括号里就是要添加的复合主键：

  ![image-20231222192052043](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222192052043.png)

- 在建完表之后，可以添加多个字段同时设置复合主键：

  ![image-20231222192520002](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231222192520002.png)

> - 复合主键只能设置一次；
> - 已经有主键或者复合主键了，就不能再设置复合主键了。

## 自增长

当设置了自增长的字段未给值时，系统会给该字段设置一个值，该值为当前列已有值的最大值+1。当无值时，默认值为 1。

在建表时字段的后面添加上 `auto_increment` 即可。

![image-20231223202228665](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231223202228665.png)

> - 自增长的字段必须是索引，通过 `desc` 查看时 Key 栏非空。
> - 自增长的字段必须是整数。
> - 一张表只能有一个自增长字段。

## 唯一键

一张表中，往往可能需要多个字段都保持唯一性，而主键虽然能保证唯一性，但一张表只允许有一个主键。于是唯一键就诞生了，唯一键允许存在多个，并且唯一键允许字段为空。

在建表时，在字段的后面加上 `unique` 即可。

![image-20231223203033728](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231223203033728.png)

## 外键







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