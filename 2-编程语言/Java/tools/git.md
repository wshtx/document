# Git

## 分支操作

- 创建分支 git branch name
- 查看分支 git branch -v
- 切换分支 git checkout name
- 合并分支
  - 先切换至被合并分支 git checkout master
  - 合并分支 git merge name
- 删除分支

## 查看历史记录

- git log 显式完整提交日志
- git log --pretty=online 每个提交只占一行
- git long --online 缩写
- git reflog 额外显式当前版本到其他版本的步数

## 版本前进后退

- 基于索引值[推荐]
  - git reset --hard hashId
- 使用^符号：只能后退，且不能将数字作为参数
  - git reset --hard HEAD^^^ 当前版本后退三步
- 使用~符号：只能后退，可以将数字作为参数
  - git reset --hard HEAD~3 当前版本后退三步

## reset命令的参数

- soft
  - 仅仅在本地库设置HEAD指针
- mixed
  - 在本地库设置HEAD指针
  - 重置暂存区
- hard
  - 在本地库设置HEAD指针
  - 重置暂存区
  - 重置工作区

## 文件比较

- `git diff file` 工作区的该文件和暂存区该文件比较
- `git diff HEAD file` 和版本库当前版本相比较
- `git diff HEAD^^ file` 和版本库回退两个版本的该文件进行比较