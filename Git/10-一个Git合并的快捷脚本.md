该脚本可以快捷合并某分支到当前分支，并忽略冲突文件(冲突文件通过读取 .gitattributes 来忽略)。

```bash
#!/bin/bash

function color(){
    c=${1}
    shift
    case ${c} in
        r)  #.红色
        echo -e "\033[1;31;1m${*}\033[0m"
        ;;
        g)  #.绿色
        echo -e "\033[0;32;1m${*}\033[0m"
        ;;
        y)  #.黄色
        echo -e "\033[0;33;1m${*}\033[0m"
        ;;
        b)  #.蓝色
        echo -e "\033[0;34;1m${*}\033[0m"
        ;;
        p)  #.紫色
        echo -e "\033[0;35;1m${*}\033[0m"
        ;;
        c)  #.青色
        echo -e "\033[0;36;1m${*}\033[0m"
        ;;
        s)  #.带闪烁黄色
        echo -e "\033[5;33;1m${*}\033[0m"
        ;;
        rs) #.带闪烁红色
        echo -e "\033[5;31;1m${*}\033[0m"
        ;;
        *)
        echo -e "${*}"
        ;;
    esac
}


if [ $# -lt 2 ]; then
    echo "sh git_merge.sh beta(合并分支) develop(被合并分支)"
    echo "eg.:"
    echo "  sh git_merge.sh beta develop # develop合并到beta"
    exit 0
fi

dstBranch=$1
srcBranch=$2
color r "merge ${srcBranch} to ${dstBranch} ..."
echo

curBranch=`git branch --show-current`
if [[ $curBranch != $dstBranch ]]; then
    color r "current branch is not ${dstBranch}!!!"
    exit 1
fi

if ! git show-ref --verify --quiet refs/heads/${srcBranch}; then
    color r "${srcBranch} not exist!!!"
    exit 1
fi

color g "pull ${srcBranch}..."
git checkout ${srcBranch}
if [[ $? != 0 ]]; then
    exit 1
fi
git pull origin ${srcBranch}
if [[ $? != 0 ]]; then
    exit 1
fi
color g "pull ${srcBranch} ok!"
echo

color g "pull ${dstBranch}..."
git checkout ${dstBranch}
if [[ $? != 0 ]]; then
    exit 1
fi
git pull origin ${dstBranch}
if [[ $? != 0 ]]; then
    exit 1
fi
color g "pull ${dstBranch} ok!"
echo

color r "start merge ${srcBranch} to ${dstBranch}..."
git merge --no-ff --no-commit ${srcBranch}
if [[ $? != 0 ]]; then
    exit 1
fi
color r "merge finish!"
echo

grep -w "merge=ours" .gitattributes | awk '{print $1}' |xargs -t -I {} git checkout ${dstBranch} -- {}

```
