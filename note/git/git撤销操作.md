



# git 撤销操作

**disk**

| command      | description                                                |
| ------------ | ---------------------------------------------------------- |
| 查看修改     | `git diff`                                                 |
| 查看状态     | `git status` -> `Changes not staged for comit`             |
| 撤销文件修改 | `git checkout <change_file>  or git restore <change_file>` |
| 提交暂存区   | git add <change_file>                                      |

**暂存区**

| command                          | description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| 查看状态                         | `git status` -> `Changes to be committed(绿色)`              |
| 从暂存区移除，但保留硬盘上的修改 | `git reset <change_file>` or `git restore --staged <change_file>` |
| 从暂存区移除，不保留硬盘上的修改 | `git checkout HEAD <change_file>`                            |
| 提交本地git                      | `git commit`                                                 |

**local**

| command                                                 | description                                      |
| ------------------------------------------------------- | ------------------------------------------------ |
| 撤销commit(保留磁盘上的修改和暂存区记录)                | `git reset --soft HEAD~1`                        |
| 撤销commit(清除暂存区记录, 只保留磁盘上的修改)          | `git reset HEAD~1` == `git reset --mixed HEAS~1` |
| 撤销commit(清除暂存区记录, 清除磁盘上的修改)            | `git reset --hard HEAD~1`                        |
| 生成新的`commitId`,将上一个`commit+`的内容变成`commit-` | `git revert HEAD`                                |
| 提交远端git                                             | `git push`                                       |

`git reset` & `git revert`:

1. `git reset`: 只能回到之前某一个commit的状态。
2. `git revert`:撤销中间任意一个commit。`git revert 70a0;(git revert HEAD~1)`

如果操作项目的分支是公共分支，只能通过 `git revert` 生成一个新的 commitId，从这个结果上撤销我们之前的修改。

1. `git revert HEAD`
2. `git push`

如果操作项目的分支是个人分支，可以通过`git reset`撤销我们之前的修改

1. `git reset --hard HEAD~1`
2. `git push -f`

