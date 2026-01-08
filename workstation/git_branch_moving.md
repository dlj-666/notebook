---
[[git]]
---
# 创建新分支并指向某个版本
===============================
git checkout-b new_branch_name <commit-hash>

# 切换到某个分支
已存在：git checkout <分支名>

# 查看所有分支
git branch -a  #这个命令会查看所有分支，包括远程的和本地的，要使用远程分支需要创建对应的本地分支，
eg:远程分支remotes/origin/develop  # 要使用远程分支：git checkout develop

# 查看所有分支及分支图
git log --oneline --graph --all

# 在旧版本创建分支提交不会覆盖原来的路，二十开一条新路

