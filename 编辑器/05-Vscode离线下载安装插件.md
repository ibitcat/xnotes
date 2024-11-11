## 批量下载插件安装包

在有网络环境的机器上，先下载好插件安装包，安装包的后缀为 `.vsix`，可以在外网机的 git bash 下运行下面的脚本：

```bash
#!/bin/bash

if [ $# -lt 2 ]; then
    echo "sh vsix.sh json配置文件 下载路径 是否全量打包"
    echo "eg.:"
    echo "    sh vsix.sh extensions.json ./download"
    exit 0
fi

extensions=$1
download=$2
isfull=$3
echo "extensions: ${extensions}"
echo "download: ${download}"
mkdir -p $download

OS_TYPE=$(uname -s)
if [[ "$OS_TYPE" == "MINGW"* ]]; then
    echo "当前运行在 Windows (Git Bash)"
    if ! command -v wget >/dev/null 2>&1; then
        echo "wget not found"
        curl -L -o /usr/bin/wget.exe https://eternallybored.org/misc/wget/1.21.4/64/wget.exe
    fi
    if ! command -v jq >/dev/null 2>&1; then
        echo "jq not found"
        curl -L -o /usr/bin/jq.exe https://github.com/jqlang/jq/releases/latest/download/jq-win64.exe
    fi
elif [[ "$OS_TYPE" == "Linux" ]]; then
    echo "当前运行在 Linux"
elif [[ "$OS_TYPE" == "Darwin" ]]; then
    echo "当前运行在 macOS"
else
    echo "未知的操作系统: $OS_TYPE"
fi

changeVsixs=""
urlPrefix="https://marketplace.visualstudio.com/_apis/public/gallery/publishers"

echo ""
echo "开始下载安装包..."
versionInfo=$(code --version)
binVer=$(echo $versionInfo | awk '{print $1}')
binHash=$(echo $versionInfo | awk '{print $2}')
binPlat=$(echo $versionInfo | awk '{print $3}')
binName="VSCodeUserSetup-${binPlat}-${binVer}.exe"
binUrl="https://vscode.download.prss.microsoft.com/dbazure/download/stable/${binHash}/${binName}"
echo "${binUrl}"

if [ ! -f "${download}/${binName}" ]; then
    wget -P ${download} -q --show-progress --progress=bar:force::noscroll ${binUrl}
    changeVsixs="${changeVsixs} ${download}/${binName}"
fi

echo ""
echo "开始下载插件..."
shopt -s lastpipe
cat ${extensions} | jq -c '.[]' | while read -r item; do
    #echo "$item"
    id=$(echo $item | jq -r '.identifier.id')
    author=$(echo ${id} | awk -F. '{print $1}')
    exname=$(echo ${id} | awk -F. '{print $2}')
    version=$(echo $item | jq -r '.version')
    platform=$(echo $item | jq -r '.metadata.targetPlatform')
    echo "$id"

    vsix="${id}-${version}@${platform}.vsix"
    url="${urlPrefix}/${author}/vsextensions/${exname}/${version}/vspackage?targetPlatform=${platform}"
    if [ "$platform" = "null" ] || [ "$platform" = "undefined" ]; then
        vsix="${id}-${version}.vsix"
        url="${urlPrefix}/${author}/vsextensions/${exname}/${version}/vspackage"
    fi

    if [ ! -f "${download}/${vsix}" ] || ! unzip -t "${download}/${vsix}" >/dev/null 2>&1 ; then
        echo "vsix=${vsix}, url=$url"
        wget -q --show-progress --progress=bar:force::noscroll -L --content-disposition ${url} -O- | gunzip -c > ${download}/${vsix}
        #curl -OLJ -N ${url} -#

        # 文件完成才打包进来
        if unzip -t "${download}/${vsix}" >/dev/null 2>&1 ; then
            changeVsixs="${changeVsixs} ${download}/${vsix}"
        fi
    else
        if [ "$isfull" = "1" ]; then
            if unzip -t "${download}/${vsix}" >/dev/null 2>&1 ; then
                changeVsixs="${changeVsixs} ${download}/${vsix}"
            fi
        fi
    fi
done
#done < <(cat ${extensions} | jq -c '.[]')

#echo "changeVsixs = ${changeVsixs}"
echo ""
echo "开始打包..."
if [ -n "$changeVsixs" ]; then
    datetime=$(date +"%Y%m%d%H%M%S")
    tar -cvf vsixs-${datetime}.tar ${changeVsixs}
fi

```

运行示例：

```bash
# 读取本机的插件配置文件，下载到download目录，并生成 vsixs.tar 打包文件
bash vsix.sh ~/.vscode/extensions/extensions.json ./download
```

## 离线环境下安装插件

先解压 vsixs.tar，在解压后的目录下执行以下 CMD，可批量安装插件：

```batch
for /f "delims=?" %i in ('dir /b /s "*.vsix" ^| sort') do (code --install-extension %~nxi)
```
