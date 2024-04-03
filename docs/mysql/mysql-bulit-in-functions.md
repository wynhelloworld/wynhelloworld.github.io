# MySQL 内置函数

## 日期函数

| 函数                               | 说明                                              |
| ---------------------------------- | ------------------------------------------------- |
| current_date()                     | 当前日期                                          |
| current_time()                     | 当前时间                                          |
| current_timestamp()                | 当前时间戳                                        |
| date(datetime)                     | 返回 datetime 参数的日期部分                      |
| date_add(date, interval expr type) | 在 date 中添加日期或时间，interval 后跟时间和单位 |
| date_sub(date, interval expr type) | 在 date 中删除日期或时间，interval 后跟时间和单位 |
| datediff(date1, date2)             | 两个日期的差，单位是天                            |
| now()                              | 当前日期时间                                      |

获得当前日期：

![image-20240403172912538](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403172912538.png)

获得当前时间：

![image-20240403172942035](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403172942035.png)

获得当前时间戳：

![image-20240403173046436](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403173046436.png)

获得时间戳中的日期：

![image-20240403173332107](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403173332107.png)

在日期的基础上加上时间：

![image-20240403173604453](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403173604453.png)

在日期的基础上减去时间：

![image-20240403173652978](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403173652978.png)

计算两个日期的差：

![image-20240403174139292](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403174139292.png)

获得当前日期时间：

![image-20240403174236406](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403174236406.png)

## 字符串函数

| 函数                                  | 说明                                                        |
| ------------------------------------- | ----------------------------------------------------------- |
| charset(str)                          | 返回 str 的字符集                                           |
| concat(str, [. . .])                  | 将 str 们拼接起来                                           |
| instr(str, substr)                    | 返回 substr 在 str 中的位置，若 str 中没有 substr，则返回 0 |
| ucase(str)                            | 将 str 的字符转换成大写                                     |
| lcase(str)                            | 将 str 的字符转换成小写                                     |
| left(str, len)                        | 返回 str 从左起 len 个字符                                  |
| length(str)                           | 返回 str 的 字节数                                          |
| replace(str, search_str, replace_str) | 将 str 中的 search_str 替换成 replace_str                   |
| strcmp(str1, str2)                    | 比较 str1 和 str2                                           |
| substring(str, pos [, len])           | 返回 str 从 pos 位置开始 len 个字符的子串                   |
| ltrim(str)                            | 去除前导空格                                                |
| rtrim(str)                            | 去除后导空格                                                |

返回 emp 表中的 ename 列的字符集：

![image-20240403175637215](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403175637215.png)

按格式输出信息：

![image-20240403175855022](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403175855022.png)

## 数学函数

| 函数                             | 说明                                           |
| -------------------------------- | ---------------------------------------------- |
| abs(number)                      | 求绝对值                                       |
| bin(decimal_number)              | 转换成 2 进制                                  |
| hex(decimal_number)              | 转换成 16 进制                                 |
| conv(number, from_base, to_base) | 将 number 从 from_base 进制转换到 to_base 进制 |
| ceiling(bumber)                  | 向上取整                                       |
| floor(number)                    | 向下取整                                       |
| format(number, decimal_places)   | 格式化，保留小数位数                           |
| mod(N, M)                        | 求 N % M                                       |
| rand()                           | 获得随机浮点数，范围 [0.0, 1.0)                |

求绝对值：

![image-20240403181531414](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403181531414.png)

向上取整：

![image-20240403181600749](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403181600749.png)

向下取整：

![image-20240403181612978](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403181612978.png)

保留 2 位小数位数（小数四舍五入）：

![image-20240403181644749](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403181644749.png)

获得随机数：

![image-20240403181706313](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403181706313.png)

## 其它函数

| 函数               | 说明                                            |
| ------------------ | ----------------------------------------------- |
| user()             | 查询当前用户                                    |
| md5(str)           | 获得 str 的 md5 码                              |
| database()         | 查询当前使用的数据库                            |
| password(str)      | 对 str 进行加密                                 |
| ifnull(val1, val2) | 如果 val1 不为 null，就返回 val1，否则返回 val2 |

查询当前用户：

![image-20240403182020297](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403182020297.png)

获得 str 的 md5 码：

![image-20240403182049976](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403182049976.png)

查询当前使用的数据库：

![image-20240403182144902](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403182144902.png)

对 str 进行加密：

![image-20240403182242363](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240403182242363.png)



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