## 使用场景
随着提交数量的增长，git 仓库的体积也会越来越大，比如一些博客系统自动部署后产生的 commit 记录，这些 commit 其实可以清空掉，当然清空后是**无法再回滚还原**的。

清空后的效果有点类似 `git clone https://xxxx/test.git --depth=1`，减少了仓库的大小。

## 操作步骤
```bash
# 创建新的孤立分支
git checkout --orphan newBranch

# 添加所有文件
git add -A

# 提交所有文件
git commit -am "commit message"

# 删除主分支
git branch -D master

# 重命名当前分支
git branch -m master

# 强制提交推送
git push -f origin master
```

注意，推送后最好删除旧仓库再重新拉一次，因为原来拉下来的仓库，还是会保留者之前的 commit 提交。