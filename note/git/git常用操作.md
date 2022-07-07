




---
# 一、安装
[官方网站](https://git-scm.com/)

## 1.1 下载
[下载地址](https://git-scm.com/downloads)

## 1.2 安装
双击下载好的 exe 一路 next 即可

# 二、使用 git 的工作流程
1. 克隆 git 项目作为工作目录
2. 在克隆下来的项目进行修改
3. 协同开发更新了文件, 就可以在本地进行文件更新
4. 在自己提交前查看修改
5. 提交修改
6. 在修改完成后, 发现有错误, 可以撤销并再次修改提交

![工作流程图](../../img/git/工作流程图.png)

# 三、git 配置相关常用命令
## 3.1 git config
查看 git 配置
```bash
$ git config --list
credential.helper=osxkeychain
user.name=kino
user.email=416595168@qq.com
core.autocrlf=input
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=https://bdgit.9zdata.cn/kino/kino-test.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
```
修改 git 配置文件
```bash
$ git config -e            # 针对当前仓库
$ git config -e --global   # 针对系统上的所有仓库
```
设置提交代码时的用户信息
```bash
$ git config --global user.name "yourUserName"     # 去掉 --global 就只对当前仓库生效
$ git config --gloabl user.email "yourEmail"       # 去掉 --global 就只对当前仓库生效
```

# 四、创建仓库相关常用命令
## 4.1 git init 
```bash
$ mkdir git-test

$ cd git-test

$ git init
Reinitialized existing Git repository in D:/work/git-test/.git/

$ ll -a
total 8
drwxr-xr-x 1 raomi 197609 0 12月 17 22:59 ./
drwxr-xr-x 1 raomi 197609 0 12月 17 22:59 ../
drwxr-xr-x 1 raomi 197609 0 12月 17 23:00 .git/
```

## 4.2 git clone
克隆一个 git 仓库到本地, 让自己能够查看该项目, 或者进行修改
```bash
$ git clone [url]
```
例如: 
```bash
$ git clone https://github.com/KinoMin/git-study.git
Cloning into 'git-study'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), 807 bytes | 24.00 KiB/s, done.

$ cd git-study
$ ll -a 
total 5
drwxr-xr-x 1 raomi 197609  0 12月 17 23:04 ./
drwxr-xr-x 1 raomi 197609  0 12月 17 23:04 ../
drwxr-xr-x 1 raomi 197609  0 12月 17 23:04 .git/
-rw-r--r-- 1 raomi 197609 19 12月 17 23:04 README.md
```
默认情况下, git 按照提供的 url 所指向的项目名称创建本地项目目录, 通常就是最后一个 `/` 之后的名字, 如果想要别的名字, 可以在命令最后添加想要的名称
```bash
$ git clone https://github.com/KinoMin/git-study.git git-study-1
Cloning into 'git-study-1'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), 807 bytes | 26.00 KiB/s, done.

$ cd git-study-1
$ ll -a 
total 5
drwxr-xr-x 1 raomi 197609  0 12月 17 23:07 ./
drwxr-xr-x 1 raomi 197609  0 12月 17 23:06 ../
drwxr-xr-x 1 raomi 197609  0 12月 17 23:07 .git/
-rw-r--r-- 1 raomi 197609 19 12月 17 23:07 README.md
```

# 五、提交与修改相关的常用命令
git 的工作就是创建和保存你的项目的快照与之后的快照进行对比
## 5.1 git add
该命令可以将文件添加到暂存区
```bash
$ git add [file1] [file2] ...
```
添加指定目录到暂存区
```bash
$ git add [dir]
```
添加当前目录下所有文件进入暂存区
```bash
$ git add .
```
例子如下:
```bash
$ touch 1.txt 2.txt 3.txt 4.txt
$ ll 
total 8
-rw-r--r--  1 kino  staff   0  7  5 18:30 1.txt
-rw-r--r--  1 kino  staff   0  7  5 18:30 2.txt
-rw-r--r--  1 kino  staff   0  7  5 18:31 3.txt
-rw-r--r--  1 kino  staff   0  7  5 18:31 4.txt
-rw-r--r--  1 kino  staff  13  7  5 18:30 README.md

# 添加 1.txt 和 2.txt 进入暂存区
$ git add 1.txt 2.txt
```

## 5.2 git status
查看在你上次提交之后是否有对文件进行再次修改。
```bash
$ git status 
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   1.txt
	new file:   2.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	3.txt
	4.txt

$ git status -s 
A  1.txt
A  2.txt
?? 3.txt
?? 4.txt

$ git commit -m "提交"
[main 26c1139] 提交
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 1.txt
 create mode 100644 2.txt

$ echo 11111 >> 1.txt
$ echo 33333 >> 3.txt
$ git add 3.txt

$ git status -s 
 M 1.txt
AM 3.txt
?? 4.txt
```
- A 表示新提交
- M 表示提交过，并且本地又修改了
- AM 表示有改动

## 5.3 git diff
比较文件的不同, 即比较文件在暂存区和工作区的差异

git diff 显示已经写入暂存区和已经被修改但尚未写入暂存区文件的区别

应用场景:
1. 尚未缓存的改动: git diff
2. 查看已经缓存的改动: git diff --cached
3. 查看缓存成功和未缓存的所有改动: git diff HEAD
4. 显示摘要而非整个diff: git diff --stat

删除原来的项目重新clone
```bash
$ rm -rf git-study
$ git clone https://github.com/KinoMin/git-study.git

$ cd git-study

$ echo 1111 >> README.md

# 查看暂未添加至缓存区的改动
$ git diff
warning: LF will be replaced by CRLF in README.md.
The file will have its original line endings in your working directory
diff --git a/README.md b/README.md
index 25df083..5cf7833 100644
--- a/README.md
+++ b/README.md
@@ -1,3 +1,3 @@
 # git-study

-test
\ No newline at end of file
+test1111

# 显示简略信息
$ git diff --stat
warning: LF will be replaced by CRLF in README.md.
The file will have its original line endings in your working directory
 README.md | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

# 查看已经添加至缓存区的改动
$ git add README.md
diff --git a/README.md b/README.md
index 25df083..5cf7833 100644
--- a/README.md
+++ b/README.md
@@ -1,3 +1,3 @@
 # git-study

-test
\ No newline at end of file
+test1111


# 查看不在缓存区 & 在缓存区的改动
$ git diff HEAD
diff --git a/README.md b/README.md
index 25df083..5cf7833 100644
--- a/README.md
+++ b/README.md
@@ -1,3 +1,3 @@
 # git-study

-test
\ No newline at end of file
+test1111
```

## git ls-files
查看暂存区的文件
```bash
$ git ls-files
1.txt
README.md
```
可选参数:
- -c: 默认
- -d: 显示删除的文件
- -m: 显示被修改过的文件
- -o: 显示没有被 git 跟踪过的文件
- -s: 显示 mode 以及对应的 Blog对象, 进而可以获取暂存区中对应文件的内容

## git cat-file -p
查看暂存区文件中的内容
```bash
$ git ls-files -s
100644 1f67fc4386b2d171e0d21be1c447e12660561f9b 0       1.txt
100644 5cf7833392d65968a05971b9923f36a1ec46d7f7 0       README.md

$ git cat-file -p  5cf7
# git-study

test1111
```

## 5.4 git commit
提交暂存区文件到本地仓库
```bash
$ git commit -m [message]
```
提交暂存区的指定文件到本地仓库
```bash
$ git commit [file1] [file2] ... -m [message]
```
-a 参数设置后, 不需要执行 git add 命令, 直接提交
```bash
$ git commit -a
```

## 5.5 git reset
回退版本

## 5.6 git rm
`git rm` 用于删除文件

1、将文件从暂存区和工作区中删除
```bash
$ git rm 1.txt 2.txt
```
2、将文件从暂存区和工作区中强制删除
```bash
# 可以加上 -f, 表示强制删除之前修改过而且 add 到暂存区的文件
$ git rm -f 1.txt 2.txt
```
3、将文件从暂存区删除，在工作区保留
```bash
git rm --cached 1.txt 2.txt
```


## 5.7 git mv 

## 5.8 git 取消 commit
1、尚未 push 的 commit
```bash
$ git reset [--soft | --mixed | --hard] <commit_id> 
```
- `--soft`: 不删除工作空间的改动代码, **撤销commit, 不撤销git add file**,如果还需要提交,直接commit即可.
- `--mixed`: 不删除工作空间的改动代码,撤销commit,撤销git add file.
- `--hard（慎用）`: 删除工作空间的改动代码,撤销commit,撤销add.

1、已经 push 的 commit