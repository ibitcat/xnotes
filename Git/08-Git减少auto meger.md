## 产生原因
Auto Merge 通常发生在本地 commit 后，想推送到远端，但是远端的进度已经被其他人更新过，此时，git会提醒你更新，当 pull 下来后，如果没有冲突，本地的提交就会和远端产生一次 Auto Merge，也就是git自动进行一次分支合并，这次合并的结果就会产生一个 commit，它有两个父亲提交。

## 如何避免
利用 git 的 pre-commit hooks，在本地 commit 前，把跟踪分支 fetch 下，让跟踪分支和远端同步，然后对比**远端**、**跟踪**和 **本地** 它们的版本，来判断本地是需要 pull 还是 push，hook 脚本如下：
```bash
#!/bin/sh

echo "git fetching"
git fetch
echo "git fetch finished"
echo ""

UPSTREAM=${1:-'@{u}'}
LOCAL=$(git rev-parse @)
REMOTE=$(git rev-parse "$UPSTREAM")
BASE=$(git merge-base @ "$UPSTREAM")

echo "Local:  $LOCAL"
echo "Remote: $REMOTE"
echo "Base:   $BASE"
echo ""

if [ $LOCAL = $REMOTE ]; then
    echo "Up-to-date"
elif [ $LOCAL = $BASE ]; then
    echo "Need to pull"
    echo "=================提交失败==============="
	echo "Error:远程端有新内容，请更到最新后后再提交内容"
    exit 1
elif [ $REMOTE = $BASE ]; then
    echo "Need to push"
else
    echo "Diverged"
fi
```

参考：[Check if pull needed in Git](https://stackoverflow.com/questions/3258243/check-if-pull-needed-in-git).