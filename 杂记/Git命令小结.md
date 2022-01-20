# Git命令小结

## 选项
```bash
# 查看git版本
git --version

# 帮助文档
git --help
```

## 命令
### init
```bash
# 在当前目录新建一个代码库
git init

# 新建一个目录，并将其初始化为git代码库
git init demo-project
```
### config
```bash
# 显示当前git的配置，进入后按Q退出
git config --list

# 编辑当前的配置文件
git config -e [--global]

# 设置提交代码时的用户信息
git config [--global] user.name ""
git config [--global] user.email ""

# 查看设置的用户信息
git config [--global] user.name
git config [--global] user.email
```
### add
```bash
# 添加指定的文件到暂存区
git add [file1] [file2]

# 添加指定的目录到暂存区，包含子目录
git add [dir]

# 添加当前目录的所有文件到暂存区
git add .
```
### rm
```bash

```
### commit
```bash
# 提交暂存区到仓库区
git commit -m ""

# 提交暂存区指定的文件到仓库区
git commit [file1] [file2] -m ""

# 提交时显示所有的diff信息
git commit -v

# 使用新一次的commit代替上一次
git commit --amend -m ""
```
### push
```bash
# 上传本地指定分支到远程仓库
git push [remote] [branch]

# 推送所有分支到远程仓库
git push [remote] --all

# 删除远程分支
git push origin --delete [branch-name]

# 提交指定tag
git push [remote] [tag]

# 提交所有tag
git push [remote] --tags

# 删除远程tag
git push origin :refs/tags/[tagName]
```
### branch
```bash
# 列出当前所有的本地分支
git branch

# 列出所有远程分支
git branch -r

# 列出所有本地分支和远程分支
git branch -a

# 新建一个分支，但是仍然停留在当前分支
git branch [branch-name]

# 删除分支
git branch -d [branch-name]

# 删除远程分支
git branch -dr [branch-name]
```
### checkout
```bash

```
### tag
```bash

```
## 参考
https://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html

https://blog.csdn.net/halaoda/article/details/78661334

https://www.git-scm.com/docs/git#_git_commands