# Git

Git 是一款版本控制管理系统。

简单明了的说 Git 的作用就是，如果一组文件被 Git 管理起来了，那么对于这一组文件的所有变动，Git 都会记录下来，以便用户能够随意切换到某一个变动的版本。

[Git 是什么？](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-Git-%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F)

## 安装

**Ubuntu**

```shell
sudo apt-get install git -y
```

**Centos**

```shell
sudo yum install git -y
```

**查看 git 的版本**

```shell
git [-v | --version]
```

## 基本操作

### 创建本地仓库

在一个目录下执行该指令

```shell
git init
```

此时目录下会存在一个`.git`目录。

### 配置基本信息

* 配置用户信息作用于当前 Git 仓库（必须做）

  ```shell
  git config [--global] user.name "Your name"
  git config [--global] user.email "Your email"
  ```

> 加`--global`选项使得配置用户信息作用于全部仓库。

* 查看配置信息

  ```shell
  git config -l
  ```

* 删除配置信息

  ```shell
  git config [--global] --unset user.name
  git config [--global] --unset user.email
  ```

> 加`--global`选项作用于删除全部仓库的用户配置信息

在创建完本地仓库以及配置好用户信息后，就能够正常使用 Git 了。

### 认识工作区、暂存区、版本库

* **工作区**：是当前目录里除了`.git`目录之外的地方。
* **暂存区**：是`.git`目录里的一个文件，它叫做`index`或者`stage`。
* **版本库**：是`git`目录。

下面这张图展示了工作区、暂存区和版本库三者之间的关系：

<img width="898" alt="image" src="https://github.com/bytenan/Git/assets/100895805/04c9a135-3279-41cd-911e-b34ce9e87681">


* 版本库中有一个`objects`目录，每次在执行`git add`或者`git commit`之后，都在`objects` 下添加一个目录，目录的名字为哈希摘要的前 2 位，目录下有个文件，文件的名字为哈希摘要的后 38 位。add 操作添加的文件的内容就是工作区里文件的内容，commit 操作添加的文件的内容里有 commit message，还有其它哈希摘要（其它目录和文件），里面分别是 commit 操作之前的暂存区里的内容。
* 暂存区中存放的是文件的 hash 值。
* `.git`下有一个`HEAD`，里面存放的是当前分支头。
* 分支头里存放的是最近一次版本的文件 hash。

### 添加文件

将工作区中的文件变动添加到暂存区。

```shell
git add [file]
```

### 提交文件

将暂存区中的文件变动提交到版本库。

```shell
git commit [file] -m "commit message"
```

> 当`file`未指定时，会将暂存区中的全部文件变动都提交到版本库。
>
>  `-m`后的提交信息要认真写，因为这个提交信息可以为以后用户进行版本回溯时提供非常重要的信息。

### 查看文件的修改信息

* 查看工作区和暂存区文件的差异

  ```shell
  git diff [file]
  ```

* 查看工作区和版本库文件的差异

  ```shell
  git diff HEAD -- [file]
  ```

### 版本回溯

#### 显示日志

* 方法一：显示的内容较详细

  ```shell
  git log
  ```

* 方法二：显示的内容较精简

  ```shell
  git log --pretty=oneline 
  ```

- 方法三：以图的形式显示日志，并且哈希值是精简的

  ```shell
  git log --graph --abbrev-commit
  ```

> 每一次 commit 都是一个版本。
>
> 显示日志时，最重要的就是 commit ID，称为版本号。

#### 进行版本回溯

```shell
git reset [--soft | --mixed | --hard] [HEAD]
```

选项说明：

* `[--soft | --mixed | --hard]`：指定回溯的区域。

  | 工作区 | 暂存区 | 版本库 |       选项        |
  | :----: | :----: | :----: | :---------------: |
  |   ——   |   ——   |  回退  |       soft        |
  |   ——   |  回退  |  回退  | mixed（默认选项） |
  |  回退  |  回退  |  回退  |       hard        |

* `[HEAD]`：指定回溯的版本。

  |      版本      |   选项一    |   选项二    |
  | :------------: | :---------: | :---------: |
  |    当前版本    |    HEAD     |   HEAD~0    |
  |    上个版本    |    HEAD^    |   HEAD~1    |
  |   上上个版本   |   HEAD^^    |   HEAD~2    |
  |  . . . . . .   | . . . . . . | . . . . . . |
  | 版本号对应版本 |   版本号    |   版本号    |

#### 版本回溯后反悔怎么办

* 方法一：若还记得版本号，那就直接用该版本号再来一次版本回溯；
* 方法二：若不记得版本号，那就执行命令`git reflog`，该命令记录了本地的每一次命令，用显示内容最前面的 16 进制数进行版本回溯即可（虽然该值很短，但可用）。

#### 版本回溯的原理

HEAD 里存的是当前分支头，版本回溯的原理就是直接修改当前分支头的指向。所以版本回溯的速度非常快。

<img width="969" alt="image" src="https://github.com/bytenan/Git/assets/100895805/2974e3b4-5e80-416a-9bf1-bd26cdbd354c">


Git 文档里说”Reset current HEAD to the specified state.“但仍不清楚，因为当 Git 仓库有多个分支时，上面的描述就不够清晰。

#### 撤销修改

* 工作区发生变动，但没有 add 和 commit 时：

  执行`git checkout -- [file]`，可以将该文件的变动撤销回上一次 add 或 commit 时。

  > git checkout --只作用于曾经 add/commit 过的文件，并且必须要指定文件名。

* 工作区发生变动，但仅执行了 add，没有执行 commit 时：

  * 方法一：

    先执行`git reset HEAD [file]`，让 file 版本回溯到当前版本，然后再执行`git checkout -- [file]`.

    > git reset HEAD 如果不指定文件名，那么将会把暂存区所有变动都回溯到当前版本。
    >
    > 但执行 git checkout - - 时仍需要指定文件名。

  * 方法二：

    执行`git reset --hard HEAD`，让工作区和暂存区都直接回溯到当前版本。

* 工作区发生变动，add 和 commit 都执行完时：

  执行`git reset --hard HEAD^`，让工作区、暂存区和版本库都回溯到上个版本。

### 删除文件

1. `git rm file`
2. `git commit -m "commit message"`

## 分支管理

### 基础命令

**查看分支**

```shell
git branch
```

**创建分支**

```shell
git branch <name>
```

**切换分支**

```shell
git checkout <name>
```

**创建并切换分支**

```shell
git checkout -b <name>
```

**删除分支**

```shell
git branch -d <name>
```

**强制删除分支**

```shell
git branch -D <name>
```

> 当分支上有未提交的内容时，-d 是删除不了分支的，只能用 -D 来删除。

### 合并分支

```shell
git merge <name>
```

比如分支 A 想要合并分支 B 的内容，那么首先要处于分支 A，然后执行 `git merge B`

上述命令默认的合并模式是 Fast-forward，在该模式下进行合并，在查看日志时，是看不出来最新的提交究竟是合并得来的还是分支自己搞得。	

![image-20231215194308861](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215194308861.png)

![image-20231215194357496](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215194357496.png)

所以建议在合并时不使用默认的合并模式，该用 no-ff 模式（no - fast forward 的缩写）:

```shell
git merge --no-ff -m "message" <name>
```

由于新增了一个 commit ，所以必须要写提交信息。

![image-20231215194725478](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215194725478.png)

![image-20231215194812479](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215194812479.png)

### 合并冲突

假设分支 A 在 test.cc 文件里新增了一行然后提交了，分支 B 也在 test.cc 文件里新增了一行然后提交了。当分支 A 合并分支 B 的时候就会发生合并冲突。

![image-20231215200127274](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215200127274.png)

![image-20231215200156823](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215200156823.png)

很清楚的可以看到，11111111111 和 22222222222 冲突了，此时进去文件，手动把冲突中不想要的给删除掉就 OK 了。

![image-20231215200348597](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215200348597.png)

然后，用 git status 查看状态

![image-20231215200426420](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215200426420.png)

重新提交，再查看状态，就发现冲突成功解决，合并完成。

![image-20231215200545672](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231215200545672.png)

## 远程操作

**克隆远程仓库**

```shell
git clone <远程仓库地址>
```

**向远程仓库推送**

```shell
git push <远程主机名> <本地分支名>:<远程分支名>
```

如果本地分支名与远程分支名相同，则等同于下面命令：

```shell
git push <远程主机名> <本地分支名>
```

**从远程仓库拉取**

```shell
git pull <远程主机名> <远程分支名>:<本地分支名>
```

如果远程分支名与本地分支名相同，则等同于下面命令：

```shell
git pull <远程主机名> <远程分支名>
```

**查看远程主机**

```shell
git remote -v
```

**忽略文件**

创建 `.gitignore` 文件。

```shell
touch .gitignore
```

将不想上传/不能上传的文件的后缀添加进去，比如 mac 老是自动生成一个 `.DS_Store` 文件，我们不想上传它，那就将这个后缀添加进去：

![image-20231216163537754](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231216163537754.png)

若不想上传 `.so` 文件，则把该后缀也添加进去

![image-20231216163650172](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231216163650172.png)

若此时，想上传某个 `b.so` 文件，有两种做法：

- 强制上传：

  ```shell
  git add -f b.so
  ```

- 修改 `.gitignore` 文件

  将 b.so 添加进去，并且前面加上 ！

  ![image-20231216164029978](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20231216164029978.png)

## 标签管理

**打默认标签**

```shell
git tag <tagname>
```

默认标签是默认打在最新的 commit id 上的。

**打指定标签**

```shell
git tag <tagname> <commit id>
```

**打标签时添加说明**

```shell
git tag -a <tagname> -m "message" [commit id]
```

**查看标签**

```shell
git tag
```

标签的列出顺序不是按照时间来排序的，而是按照字母排序的。

**查看标签详细信息**

```shell
git show <tagname>
```

**删除标签**

```shell
git tag -d <tagname>
```

**推送单个标签**

```shell
git push origin <tagname>
```

**推送全部标签**

```shell
git push origin --tags
```

**删除本地标签**

```shell
git tag -d <tagname>
```

**删除远程标签**

先删除本地标签，然后再推送

```shell
git push <远程主机名> :refs/tags/<tagname>
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
