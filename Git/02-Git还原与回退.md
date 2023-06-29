## Git 流程图

图取自网络。
![git 流程图](../images/git-02-01.webp)

## Git 还原
**还原**，一般指还原**工作区**的文件修改。使用 `git checkout` 可以还原工作区被修改的文件。

不同情况下，还原操作也不同。

- 修改了文件，但未加入索引
```
git checkout -- test.txt    // 从索引还原 test.txt 文件到工作区
git checkout -- *           // 从索引换到所有文件到工作区
```

- 修改了文件，且加入了索引，但未提交

    还原的核心是**如何还原索引**，并从索引检出到工作区。

    方法一：`git checkout HEAD -- test.txt`，即将文件从 HEAD 还原到索引，并从索引检出到工作区。这种方式还可以还原指定提交的文件到工作区，例如还原文件到上一次提交。
    ```
    git checkout HEAD -- test.txt  // 从HEAD还原文件修改
    git checkout HEAD^ -- test.txt // test.txt 还原到上一次提交
    ```

    方法二：借助 `git reset` 来实现。
    ```
    git reset HEAD              // 重置索引
    git checkout -- test.txt    // 从索引检出到工作区
    ```

- 修改了文件且已提交

    方法一：`git checkout HEAD^ -- test.txt` 直接从 HEAD^ 还原，这种还原方式可以保留上一次 commit 信息。

    方法二：同样借助 `git reset`，但是会丢失上一次的 commit 信息。
    ```
    git reset HEAD^             // 重置索引到上一次提交
    git checkout -- test.txt    // 检出文件
    ```

## Git 回退

### 本地分支回退
**回退**，一般用于撤销提交，使用的是 `git reset` 命令，它能修改 HEAD 指针，另外，可以通过参数来控制修改索引和工作区。其参数如下：

- `mixed`，重置HEAD，重置 index，保留工作区（*默认参数*）

    一般用于撤销提交，例如发现提交的部分代码出现错误，可以先回退到上一个版本 ` git reset HEAD^`，编辑后再次提交即可。

- `soft`，重置HEAD，保留 index，保留工作区

    一般用来修改 commit 信息，例如提交的消息有错别字想修改正确，可以如下操作：
    ```
    git reset --soft HEAD^
    git commit -m "xxxx"
    ```

    当然也可以撤销 git reset soft head操作，先通过git reflog找到上一次的历史提交记录id，然后 `git reset --hard [id]`。

- `hard`，重置HEAD，重置 index，重置工作区

    一般用于强制回退版本，且大部分情况都要重置远程仓库。
    ```
    git reset --hard [id]   // 回退到指定commit id
    git push -f             // 强制推送到远端
    或者
    git push -f -u origin develop // 强制推送到指定的远端分支
    ```

    需要注意的是，hard reset 后只影响了本地仓库 A，其他的本地仓库 B 也同样需要 hard reset，当仓库 B 拉取被重置的远端分支时，并不会合并或rebase任何东西，要特别注意，否则会导致 A 的回退操作被撤销，因为 B 又把 HEAD 推送到远端了，因此需要在 B 执行 `git reset --hard origin/master`。


### 远程分支回退
`git revert` 可以用于回退远程分支，它的回退方式与 `git reset` 有区别:

- git reset，会真正把分支的指针往回退，回退后之前的提交其实被隐藏了，并未删除，如果回退前记录了 commit id，依然是可以还原回来的
- git revert，它会创建一个新的提交，这个提交用于“撤销”指定的提交，相当于对指定提交做了一次 “undo”，然后提交，类似于某次提交对文件增加了一行，为了撤销这次提交，又再次删除这一行并提交一次，以此达到撤销的目的，分支指针是一直往前走的。

git revert 命令如下：
- `git revert <commit-id>`：撤销某个提交

    示例：
    ```bash
    # 撤销最后一次提交
    git revert HEAD

    # 撤销指定提交
    git revert 03adf6b

    # 范围撤销 range revert
    # 例如： A -> B -> C -> D(HEAD)
    # 撤销后： A -> B -> C -> D -> B' -> C' -> D'(HEAD)
    git revert B^...D
    ```
- `git revert -n|--no-commit <commit-id>`：撤销某个提交，但执行命令后不进入编辑界面，也就是不会自动提交文件，需要手动提交

在 revert 一个 merge 提交时要注意，因为一个合并提交是有两个 parent 提交的，如果直接使用 `git revert <commit-id>`，git 是不知道撤销哪条分支上的内容，此时就需要添加 `-m` 参数来指定谁作为**主线**，而另一条分支的提交将被撤销。`-m`接受一个数字作为参数，参数值为 1 或者 2。例如保留主分支提交，就应该设置主分支为主线，操作如下：
```
git revert -m 1 03adf6b
```

参考：[【git revert】使用以及理解（详解）](https://blog.csdn.net/allanGold/article/details/111372750)。

## git diff 命令
- `git diff`，比较的是工作区和暂存区的差别
- `git diff --cached`，比较的是暂存区和版本库的差别
- `git diff HEAD`，可以查看工作区和版本库的差别