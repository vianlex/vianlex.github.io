# Git 学习笔记


## 创建仓库

```bash
# 本地初始化仓库
git init 

# 关联远程仓库（注意需要在 github 上先创建远程仓库）
git remote add origin git@gitee.com:vianlex/spring-cloud-alibaba-example.git

# 同步远程仓库的修改
git pull origin master

# 提交本地仓库代码
git add . 
git commit -m "初始化仓库代码"

# 将本地仓库代码推送远程仓库
git push origin master

# 如果存在冲突文件，想丢弃远程文件，可以通过 -f 强制覆盖推送
git push origin master -f

```


## 查看分支

```bash
# 查看本地分支
git branch

# 查看所有分支（包括远程）
git branch -a

# 查看分支详情（最后提交信息）
git branch -v

# 查看已合并到当前分支的分支
git branch --merged

# 查看未合并到当前分支的分支
git branch --no-merged
```

## 切分分支

```bash
# 切换到已存在的分支
git checkout <branch-name>
# 或（Git 2.23+）
git switch <branch-name>

# 切换到上一个分支（快速切换）
git checkout -
```

## 创建分支

1、创建分支但不切换

```bash
# 创建新的分支，从当前所在分支创建新分支，新分支包含当前分支的所有提交历史
git branch <branch-name> 

# 从本地的某个提交点，创建新的分支
git branch <branch-name> <commit-hash>
# 从远程的某个提交点，创建新的分支
git branch <branch-name> origin/<commit-hash>


# 从本地标签，创建新的分支
git branch <branch-name> <tag-name>
# 从远程标签，创建新的分支
git branch <branch-name> origin/<tag-name>
```

2、创建并切换到新的分支

```bash
# 创建并切换到新的分支
git checkout -b <branch-name>
# 或（Git 2.23+推荐）
git switch -c <branch-name>
```

3、分支的命名规范

- 功能分支：feature/xxx, feat/xxx
- 修复分支：bugfix/xxx, fix/xxx, hotfix/xxx
- 发布分支：release/xxx
- 开发分支：develop
- 文档分支：docs/xxx
- 重构分支：refactor/xxx
  

## 删除分支

```bash
# 删除本地已合并分支
git branch -d <branch-name>

# 强制删除未合并分支
git branch -D <branch-name>

# 删除远程分支
git push origin --delete <branch-name>
```

## 合并分支

```bash

# 将 dev-branch 分支合并到当前分支
git merge dev-branch

# 只合并分支的某个文件的更改，到当前分支
git checkout --patch dev-branch file.txt


```


## 撤消提交