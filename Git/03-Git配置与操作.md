## Git 的配置文件在哪里
在 Windows 系统下，Git 的全局配置文件路径为 `%userprofile%/.gitconfig`。在 Linux 系统下，路径为 `$HOME/.gitconfig`。

每个 git 仓库也可以有自己的配置，配置文件路径在项目根目录的 `.git/config` 中。


## 定制git log
输入以下命令：
```
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
```

显示效果如下：
```
*   4c9c2ac - Merge branch 'lvzixun-self_skynet' (3 years, 5 months ago) <Cloud Wu>
|\
| * bfbbe91 - turn off ltls by default (3 years, 5 months ago) <Cloud Wu>
| *   eda9f77 - Merge branch 'self_skynet' of https://github.com/lvzixun/skynet into lvzixun-self_skynet (3 years, 5 months ago) <Cloud Wu>
| |\
| | * 7746193 - add https client and server support by bios (3 years, 5 months ago) <zixun>
* | |   7cb4bbb - Merge branch 'cluster' (3 years, 5 months ago) <Cloud Wu>
|\ \ \
| * | | 1c3b563 - add wait queue (3 years, 5 months ago) <Cloud Wu>
| * | | 000fa2b - multi cluster sender (3 years, 5 months ago) <Cloud Wu>
* | | |   2c247b5 - Merge branch 'sharedk' (3 years, 5 months ago) <Cloud Wu>
|\ \ \ \
| * | | | 3f38a71 - Shared K (3 years, 5 months ago) <Cloud Wu>
| |/ / /
* | / / ea1affd - fix #962 (3 years, 5 months ago) <Cloud Wu>
| |/ /
|/| |
* | | 9ffdf2d - Update skynet_daemon.c (3 years, 5 months ago) <Dalton>
|/ /
* | 1d4308f - Fix memory leak, See #952 (3 years, 6 months ago) <Cloud Wu>
* | 48e5c36 - volatile for signal, see #950 (3 years, 6 months ago) <Cloud Wu>
|/
* 1422cae - Consider __pairs, see #942 (3 years, 6 months ago) <Cloud Wu>
* 9e16b04 - dns reInit bug (3 years, 7 months ago) <Naix>
* 0fbbd5f - type can be DATA (3 years, 7 months ago) <Cloud Wu>
* 6eb9b64 - remove mysqlaux.c (3 years, 7 months ago) <Cloud Wu>
* d71c8f8 - remove new_tab (3 years, 7 months ago) <Cloud Wu>
```

## 指定合并规则

假设当前在 beta 分支，将 develop 分支代码合并到 beta，Git 可以自定义合并时的冲突出来策略。

首先，在仓库根目录(分支为`beta`)创建 `.gitattributes` 文件，在该文件中列出合并文件过滤规则，例如：
```
server/src/config/** merge=ours
server/version.txt merge=ours
.gitignore merge=ours
.gitattributes merge=ours
```
上面的每一行都指定了文件的合并规则，`merge=ours`表示**合并冲突**时使用我的(beta)，不合并对方分支(develop)的内容。

然后，在该 **仓库根目录** 执行命令：`git config merge.ours.driver true`（可以在根目录的 `.git` 目录查看本仓库的配置文件 `config`）。

**注意:** 上面的合并规则只适用于合并时产生了冲突的文件，如果文件没有冲突，文件依然会被合并，参见[Git文档](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes#_merge_strategies)。

## 合并排除指定文件
假设有两个分支 `beta` 和 `develop`，且两个分支都有自己特定的配置文件 `common.config`，当 develop 合并到 beta 时，不想将 common.config 合并过来
，当文件有冲突时可以使用上面[指定合并规则](#指定合并规则)来达到目的；但是当文件无冲突时，正确的操作应该如下：
```bash
git checkout beta
git merge --no-log --no-ff --no-commit develop
git checkout beta -- common.config
git add common.config
git commit -a -m "merge develop into beta"
git push origin beta
```

- `--no-commmit`，不要让git在合并无冲突时自动提交，若自动提交会产生一条`Merge branch ‘develop’ into beta`的提交记录
- `--no-ff`，禁止git快进(Fast-Forward)，具体作用参考[Git 合并时 --no-ff 的作用](https://blog.csdn.net/zombres/article/details/82179122)

为了同时达到在冲突和无冲突两种情况下，都能排除指定文件合并，可以写一个命令配合 `.gitattributes` 文件的 merge 规则来实现。
```
# 将上面单个文件的checkout 换成下面的命令
#grep -w "merge=ours" .gitattributes | awk '{if (system("test -f " $1)==0) print $1}' | xargs -I {} git checkout beta -- {}
grep -w "merge=ours" .gitattributes | awk '{print $1}' | xargs -I {} git checkout beta -- {}
git commit -a -m "merge develop into beta"
```

## 检查本地分支是否存在
```bash
if ! git show-ref -q --verify -- "refs/heads/$branch"; then
    echo "branch $branch does not exist"
fi
```