# MySQL 数据类型

## 数值类型

**布尔类型**

```
BOOL
```

> 0 表示假，1 表示真。

演示：

![image-20231220164835580](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231220164835580.png)

> MySQL 中并没有真正的 bool 类型，它是用 tinyint(1) 来代替的。
>
> tinyint(1) 中的 1，表示最短显示长度。假设一个字段类型是 tinyint(3)，并且设置了 zerofill，那么 2 就显示为 002。

**位类型**

```
BIT(M)
```

> M 指定位数，默认值 1, 范围 [1, 64]。

演示：

![image-20231220164457106](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231220164457106.png)

> 位类型用 ASCII 来进行显示。

**整数类型**

| 类型                   | 大小   | 有符号范围        | 无符号范围   |
| ---------------------- | ------ | ----------------- | ------------ |
| `TINYINT [UNISGNED]`   | 1 字节 | [-2^7^, 2^7^-1]   | [0, 2^8^-1]  |
| `SMALLINT [UNISGNED]`  | 2 字节 | [-2^15^, 2^15^-1] | [0, 2^16^-1] |
| `MEDIUMINT [UNISGNED]` | 3 字节 | [-2^23^, 2^23^-1] | [0, 2^24^-1] |
| `INT [UNISGNED]`       | 4 字节 | [-2^31^, 2^31^-1] | [0, 2^32^-1] |
| `BIGINT [UNISGNED]`    | 8 字节 | [-2^63^, 2^63^-1] | [0, 2^64^-1] |

**小数类型**

| 类型                          | 大小          | 精度                      |
| ----------------------------- | ------------- | ------------------------- |
| `FLOAT [(M, D)] [UNSIGNED]`   | 4 字节        | 单精度浮点数（7 位精度）  |
| `DOUBLE [(M, D)] [UNSIGNED]`  | 8 字节        | 双精度浮点数（15 位精度） |
| `DECIMAL [(M, D)] [UNSIGNED]` | 依赖于 M 和 D | 超高精度定点小数          |

> M 表示显示长度，D 表示小数点后的位数。
>
> 在 DECIMAL 中，M 范围是 [1, 65]，D 范围是 [1, 30]。M 默认为 10，D 默认为 0。
>
> float (4, 2) 的表示范围为 [-99.99, 99.99]

![image-20231220170826989](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231220170826989.png)

> 真正存入的是经过四舍五入之后的数值，若经过四舍五入之后，该数值在范围内就能存入，不在范围内就报错。

## 字符串类型

| 类型         | 大小（bytes） | 用途         |
| ------------ | ------------- | ------------ |
| `CHAR(M)`    | [0, 2^8^-1]   | 定长字符串   |
| `VARCHAR(M)` | [0, 2^16^-1]  | 变长字符串   |
| `TINYTEXT`   | [0, 2^8^-1]   | 文本字符串   |
| `TEXT`       | [0, 2^16^-1]  | 文本字符串   |
| `MEDIUMTEXT` | [0, 2^24^-1]  | 文本字符串   |
| `LONGTEXT`   | [0, 2^32^-1]  | 文本字符串   |
| `TINYBLOB`   | [0, 2^8^-1]   | 二进制字符串 |
| `BLOB`       | [0, 2^16^-1]  | 二进制字符串 |
| `MEDIUMBLOB` | [0, 2^24^-1]  | 二进制字符串 |
| `LONGBLOB`   | [0, 2^32^-1]  | 二进制字符串 |

char 和 varchar 括号里的 M 的单位是字符，而不是字节。

char 是定长字符串，举例当 M 为 8 时，该字段的大小就是 8 个字符大小（无论实际是否够 8 个字符）。

varchar 是变长字符串，举例当 M 为 8 时，该字段的大小最大为 8 个字符大小，实际大小为实际的字符大小+记录长度所占的大小。（varchar 中会用 1～3 个字节来记录实际存储的字节长度）

M 的范围和表的编码密切相关。

- 在 utf8 编码下，一个字符占用三个字节，所以 varchar 中 M 的范围是 [1, 65532/3]

- 在 bgk 编码下，一个字符占用两个字节，所以 M 的范围是 [1, 65532/2]

## 日期和时间类型

| 类型        | 大小（bytes） | 格式                |
| ----------- | ------------- | ------------------- |
| `DATE`      | 3             | YYYY-MM-DD          |
| `TIME`      | 3             | HH:MM:SS            |
| `YEAR`      | 1             | YYYY                |
| `DATETIME`  | 8             | YYYY-MM-DD HH:MM:SS |
| `TIMESTAMP` | 4             | YYYY-MM-DD HH:MM:SS |

## 枚举和集合类型

| 类型   | 用途                              |
| ------ | --------------------------------- |
| `ENUM` | 每个字段可以从 ENUM 中选择一个值  |
| `SET`  | 每个字段可以从 SET 选择零或多个值 |

语法：

```
enum('选项1', '选项2', '选项3', ...)
set('选项1', '选项2', '选项3', ...)
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