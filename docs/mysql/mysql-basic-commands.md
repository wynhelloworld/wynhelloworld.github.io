# MySQL 基础命令

MySQL 命令分为四种：

- DDL (Data Definition Language) 数据定义语言，用来维护数据的结构。

  代表命令：`create`、`alter`、`drop`

- DML (Data Manipulation Language) 数据操纵语言，用来操作数据。

  代表命令：`insert`、`delete`、`update`

- DQL (Data Query Language) 数据查询语言，用来查询数据。

  代表命令：`select`、`from` 、`where`

- DCL (Data Control Language) 数据控制语言，主要负责权限管理和事务。

  代表命令：`grant`、`revoke`、`commit`

## 查看基础配置

**查看存储引擎**

```
SHOW ENGINES;
```

**查看数据库支持的字符集**

```
SHOW CHARSET;
```

**查看数据库支持的字符集校验规则**

```
SHOW COLLATION;
```

**查看数据库默认的字符集和字符集校验规则**

```
SHOW VARIABLES LIKE 'character_set_database';
SHOW VARIABLES LIKE 'collation_database';
```

**查看连接情况**

```
SHOW PROCESSLIST;
```

## 操纵数据库

**查看数据库**

```
SHOW DATABASES;
```

**查看创建数据库时的语句**

```
SHOW CREATE DATABASE database_name;
```

**创建数据库**

```
CREATE DATABASE [IF NOT EXISTS] database_name [CREATE_SPECIFICATION [CREATE_SPECIFICATION ...]];

CREATE_SPECIFICATION:
	[DEFAULT] CHARSET SET charset_name
	[DEFAULT] COLLATE collation_name
```

> 若没有 CREATE_SPECIFICATION，则以配置文件为准。

**修改数据库**

``` 
ALTER DATABASE database_name [ALTER_SPECIFICATION [ALTER_SPECIFICATION ...]];

ALTER_SPECIFICATION:
	[DEFAULT] CHARSET SET charset_name
	[DEFAULT] COLLATE collation_name
```

> 对数据库的修改，主要指的是修改数据库的字符集和字符集校验规则。

**删除数据库**

```
DROP DATABASE [IF EXISTS] database_name;
```

**备份数据库**

```
mysqldump [OPTIONS] -B database_name > path

OPTIONS:
	-h 
	-P
	-u
	-p
	...
```

示例：

```
myqsldump -u root -p 123456 -B mydb > /home/wyn/mydb.sql
```

**还原数据库**

```
SOURCE path;
```

示例：

```
SOURCE /home/wyn/mydb.sql;
```

**同时备份多个数据库**

``` 
mysqldump [OPTIONS] -B database_name [database_name ...] > path

OPTIONS:
	-h 
	-P
	-u
	-p
	...
```

示例：

```
mysqldump -u root -p 123456 -B mydb1 mydb2 > /home/wyn/
```

**备份数据库中的某张表**

```
mysqldump [OPTIONS] database_name table_name > path

OPTIONS:
	-h 
	-P
	-u
	-p
	...
```

示例：

```
mysqldump -u root -p 123456 mydb mytable > mytable.sql
```

## 操纵表

**创建表**

```
CREATE TABLE table_name (
		filed1 data_type,
		filed2 data_type,
		filed3 data_type
) [CREATE_SPACIFICATION [CREATE_SPACIFICATION ...]];

CREATE_SPACIFICATION:
	CHARSET SET charset_name
	COLLATE collation_name
	ENGINE engine_name
```

> 若没有 CREATE_SPACIFICATION，则以配置文件为准。

**查看表结构**

```
DESC table_name;
```

**修改表结构**

```
ALTER TABLE table_name OPTIONS (filed data_type [filed data_type ...]);

OPTIONS:
	ADD
	MODIFY
	DROP
	CHANGE
	RENAME
	...
```

示例：

- 往 user 表中添加一列名为 picture_path，排在 birthday 的后面：

  ```
  ALTER TABLE user ADD (picture_path VARCHAR(100) [COMMENT '图片路径'] AFTER birthday);
  ```

- 修改 user 表中的 name 列，将其长度修改为 60：

  ```
  ALTER TABLE user MODIFY name VARCHAR(60);
  ```

- 删除 user 表中的 password 列：

  ```
  ALTER TABLE user DROP password;
  ```

- 修改 user 表中的 name 列名字为 username：

  ```
  ALTER TABLE user CHANGE name username VARCHAR(60);
  ```

- 修改 user 表的名字为 employee：

  ```
  ALTER TABLE user RENAME TO employee; 
  ```

**删除表**

```
DROP [TEMPORARY] TABLE [IF EXISTS] table_name [table_name ...];
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