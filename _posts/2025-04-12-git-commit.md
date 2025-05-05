---
title: git中如何修改commit
layout: post
---

# 修改最新提交

改变当前分支最近一次提交的最简单方法之一是使用git commit --amend，最频繁的用途是在刚做出一个提交后修改录入错误。

## 修改刚完成提交的提交信息

执行以下命令，会弹出编辑器会话，可以修改里面的提交信息。修改提交信息，保存退出即可。

```shell
git commit --amend
```


示例：

```shell
❯ echo foo >> master_file
❯ git add master_file
❯ git commit
❯ git show-branch --more=5
[master] init file master_file
```

以上是刚完成的一次提交。希望将  “init file master_file”修改为 “init file master_file.”


执行以下命令，会弹出编辑器会话，将 “init file master_file”修改为 “init file master_file.” ，保存退出，就完成了提交信息的修改。

```shell
git commit --amend
```

通过下面的结果可以看出，并没有多出新的commit。

```shell
❯ git show-branch --more=5
[master] init file master_file.
```

## 修改刚完成提交的提交内容

```shell
❯ echo foo. >> master_file
❯ git add master_file
❯ git commit
❯ git show-branch --more=5
[master] init file master_file.
```

以上是刚完成的一次提交。希望修改master_file中的文件

```shell
❯ git show-branch --more=5
[master] init file master_file.
❯ cat master_file
foo.
❯ echo "more foo" >> master_file
❯ echo "even more foo" >> master_file
❯ git add master_file
❯ git commit --amend
[master d68d6ed] init file master_file.
 Date: Sat Apr 12 19:41:26 2025 +0800
 1 file changed, 3 insertions(+)
 create mode 100644 master_file
❯ git show-branch --more=5
[master] init file master_file.
❯ cat master_file
foo.
more foo
even more foo
```



# 将刚提交的多个git commit合并为一个commit

```shell
❯ git show-branch --more=5
[master] init file master_file.
❯ cat master_file
foo.
❯ echo "more foo" >> master_file
❯ git commit master_file
[master 31b2567] add more foo to master_file
 1 file changed, 1 insertion(+)
❯ echo "even more foo" >> master_file
❯ git commit master_file
[master 31779b8] add "even more foo" to master_file
 1 file changed, 1 insertion(+)
❯ git show-branch --more=5
[master] add "even more foo" to master_file
[master^] add more foo to master_file
[master~2] init file master_file.
❯ git reset master~2
Unstaged changes after reset:
M	master_file
❯ git show-branch --more=5
[master] init file master_file.
❯ git add master_file
❯ git commit
[master 10e9d28] update foo to master_file
 1 file changed, 2 insertions(+)
❯ git show-branch --more=5
[master] update foo to master_file
[master^] init file master_file.
```


在提交多个commit后，想将最近提交的几个commit合并为一个；

关键命令 git reset master～2 ，解释：使用 git reset命令，默认参数是--mixed，head指向指定的提交，索引也跟着改变，适应新的提交树，工作区不会改变，master~2是要修改的commit的父提交；

通过git add master_file，将master_file加入索引，再执行git commit 生成一个新的提交。



## git reset 选项影响



| 选项    | HEAD | 索引 | 工作目录 |
| ------- | ---- | ---- | -------- |
| --soft  | 是   | 否   | 否       |
| --mixed | 是   | 是   | 否       |
| --hard  | 是   | 是   | 是       |

