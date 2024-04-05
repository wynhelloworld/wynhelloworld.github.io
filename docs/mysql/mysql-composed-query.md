# MySQL 复合查询

## 多表查询

在实际开发中，数据往往来自多个表，所以需要用到多表查询。

**案例：**

显示员工姓名、工资以及所在部门的名字：

![image-20240405172812618](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405172812618.png)

显示部门号为 10 的部门名字、员工姓名和工资：

![image-20240405173401075](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405173401075.png)

显示各个员工的姓名、工资和工资级别：

![image-20240405173634031](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405173634031.png)

## 自连接

自连接指的是在同一张表中连接查询。

**案例：**

显示员工 FORD 的上级领导的编号和姓名。分别使用自查询和多表查询。

子查询：

![image-20240405174340785](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405174340785.png)

多表查询：

![image-20240405174711298](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405174711298.png)

## 子查询

#### 单行子查询

子查询返回的结果是单列单行的。

**案例：**

显示与 SMITH 同一部门的员工：

![image-20240405175316052](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405175316052.png)

#### 多行子查询

子查询返回的结果是单列多行的。

**案例：**

- `in` 关键字：查询和 10 号部门的工作岗位相同的员工的姓名、岗位、工资和部门号，但是不包含 10 号部门自己：

  ![image-20240405180059474](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405180059474.png)

- `all` 关键字：查询工资比 30 号部门的所有员工的工资高的员工的姓名、工资和部门号：

  ![image-20240405180351309](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405180351309.png)

- `any` 关键字：查询工资比 30 号部门的任意员工的工资高的员工的姓名、工资和部门号（包含自己部门的员工）：

  ![image-20240405180644386](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405180644386.png)

#### 多列子查询

子查询返回的结果是多列的。

**案例：**

查询和 SMITH 的部门和岗位完全相同的所有员工，不包含 SMITH 本人：

![image-20240405181000820](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405181000820.png)

#### 在 from 子句中使用子查询

**案例：**

显示每个高于自己部门平均工资的员工的姓名、部门号、工资和平均工资：

![image-20240405181742707](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405181742707.png)

显示每个部门工资最高的人的姓名、部门号和工资：

![image-20240405182314150](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405182314150.png)

显示每个部门的信息（部门名字、部门号和地址）和人员数量（子查询和多表查询）：

子查询：

![image-20240405182831307](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405182831307.png)

多表查询：

![image-20240405183308923](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405183308923.png)

#### 合并查询

在实际应用中，可以使用 union 或者 union all 来合并多个 select 的结果。

**案例：**

将工资大于 2500 或者职位是 MANAGER 的人找出来：

- union：该操作符用于取得两个结果集的并集，并且自动去重。

  ![image-20240405184420396](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405184420396.png)

- union all：该操作符用于取得两个结果集的并集，不去重。

  ![image-20240405184451490](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240405184451490.png)



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