在 fork 一个项目后，即想要同步原作者的更新，又希望保留自己的修改，可以通过下面的步骤来实现：

1. 将原作者仓库添加为上游仓库
   fork 后，clone 自己的仓库，然后进入 git 仓库，执行如下命令：

```bash
# upstream 也可以修改成其他的名字
git remote add upstream https://github.com/cloudwu/skynet.git
git remote -v
```

2. 拉去上游仓库的更新

```bash
git fetch upsteam
```

3. 将上游仓库的更新合并到本地分支

```bash
git checkout master
git merge upstream/master
```

如果合并有冲突，需要手动处理冲突并提交。

```
git add <冲突的文件>
git commit -m "merge upstream/master"
```

4. 推送的自己的远程仓库

```bash
git push origin master
```
