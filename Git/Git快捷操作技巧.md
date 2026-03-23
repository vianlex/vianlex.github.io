# Git 快捷操作技巧

1. ` git commit -am` 快速提交
   
```bash

git add .
git commit -m "hello github"

# 等价于 

git commit -am "hello git"

```

2. ` git config --global alias` 定义命令别名

```bash

git config --global alias.ac "commit -am"

git ac "hello alias"  

# 等价于

git commit -am "hello alias"

```

3. ` git  commit --amend ` 重新修改提交，注意只有未 push 时才有效

```bash

git commit -m "Hello Wrold"

# 比如提交的单词 Wrold 拼写错了，则可以利用以下命令重新修改提交信息 
git commit --amend -m "Hello World"

# 如果只是把新修改的文件合并到上一次提交里面，不想修改信息，则使用 --no-edit
git commit --amend --no-edit

```

4. ` git revert ` 恢复到某个提交点

```
# 将当前分支恢复到到某个提交点
git revert commit-id 

```

