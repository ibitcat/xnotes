**分支保护**可以对分支进行保护，禁止其他人随意推送，例如在发版本时先对分支进行锁定，防止在发布期间有新的推送而污染分支。

>这个功能并不是 Git 原生功能，而是各种 Git 版本控制工具提供的功能，例如 Gitlab、Gitea 等都提供了分支保护。

可以利用 Git 版本控制工具提供的 API 来实现自动化锁定/解锁分支。

下面脚本以 Gitlab 为例，实现了分支查询、锁定、解锁等操作：
```bash
function _gitlabLock(){
    projId=$1
    accessToken=$2
    branch=$3
    lockop=$4

    api=http://gitlab.test.com/api/v4/projects/${projId}/protected_branches

    #type jq >/dev/null 2>&1
    declare -A dict
    dict=(["0"]="解锁" ["1"]="锁定")
    if [[ "$lockop" == "1" ]]; then
        # lock
        opName=${dict["$lockop"]}
        echo "正在${opName}仓库${projId}的分支${branch}"
        lockUrl="${api}?name=${branch}&push_access_level=40&merge_access_level=40&unprotect_access_level=40"
        lockRes=$(curl --request POST --header "PRIVATE-TOKEN: ${accessToken}" --silent ${lockUrl})
        if [[ -x "$(command -v jq)" ]]; then
            echo $lockRes | jq .
        else
            echo $lockRes
        fi
    elif [[ "$lockop" == "0" ]]; then
        # unlock
        opName=${dict["$lockop"]}
        echo "正在${opName}仓库${projId}的分支${branch}"
        unlockUrl="${api}/${branch}"
        unlockRes=$(curl --request DELETE --header "PRIVATE-TOKEN: ${accessToken}" --silent ${unlockUrl})
        echo $unlockRes
    else
        echo "正在查询仓库${projId}的分支${branch}"
        queryUrl="${api}/${branch}"
        queryRes=$(curl --header "PRIVATE-TOKEN: ${accessToken}" --silent ${queryUrl})
        echo $queryRes
    fi
}

# _gitlabLock 仓库id accessToken 分支名称 操作类型(1=锁定;0=解锁;无参数=查询)
_gitlabLock 100 xxxxxxxx $1 $2
```