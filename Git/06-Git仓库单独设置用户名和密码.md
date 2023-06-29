## git 存储密码
git 存储密码有两种方式：
- 临时存储

    ```bash
    # 密码默认存储15分钟
    # --global 表示全局配置，配置文件在当前用户根目录，如 `/home/xxx/.gitconfig`
    git config --global credential.helper cache

    # 为单个git仓库设置密码存储模式
    git config credential.helper cache

    # 设置密码到期时间为一小时
    git config credential.helper 'cache --timeout=3600'
    ```

- 长期存储

    ```bash
    # 全局存储，默认的密码存储文件为 ~/.git-credentials
    git config --global credential.helper store

    # 为单个git仓库设置永久密码存储
    git config credential.helper store

    # 指定密码存储文件
    git config credential.helper 'store --file .git/.my-credentials'
    ```

- clone时指定用户名和密码(推荐)

    ```bash
    # 示例
    git clone http://yourname:password@git.xxx.com/name/project.git -b master
    ```
    使用这种方式clone仓库后，在仓库的 `.git/config` 文件中的 `[remote "origin"]`段的 url 就会带上用户名和密码，例如：
    ```
    [remote "origin"]
        url = http://yourname:password@git.xxx.com/name/project.git
        fetch = +refs/heads/*:refs/remotes/origin/*
    ```

## 单独设置用户名和密码
当 git 仓库的配置没有设置用户名和密码时，就去读取全局配置，有两种种方法可以为git仓库单独设置用户信息和密码存储，首先要确保 `.git/config` 配置文件中 `[remote "origin"]` 的 url 带有远程仓库的账号密码，它关系到推送使用到的用户。

- 方法一：对git仓库进行单独配置

    ```bash
    # 进入仓库目录
    cd [git仓库目录]

    # 设置用户名(提交记录显示的用户名称)
    git config user.name yourname

    # 设置用户email
    git config user.email youername@email.com

    # 设置密码存储
    git config credential.helper 'store --file .git/.my-credentials'

    # 编辑 .my-credentials，填入账号密码信息
    http://yourname:password@git.xxx.com
    ```

- 方法二：在全局配置中使用`includeIf`

    编辑全局配置文件 `~/.gitconfig`，增加条件包含配置项：
    ```
    [includeif "gitdir:/data/gitrepo/"]
        path = ~/.gitconfig-xxx
    ```

    新建配置文件 `~/.gitconfig-xxx`，然后填入指定的用户信息，如下
    ```
    [user]
        name = testname
        email = test@email.com
    [credential]
        helper = store --file .git/.my-credentials
    ```

    编辑 `.git/.my-credentials`，填入账号密码。

## 参考
- [Git 工具 - 凭证存储](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8)
- [Use multiple Git accounts from a single Linux machine](https://www.attosol.com/manage-multiple-git-accounts/)