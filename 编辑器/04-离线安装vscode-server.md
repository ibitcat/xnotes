使用 Vscode 远程连接服务器，需要在服务器上安装 `vscode-server`，如果有外网访问权限则会自动下载 `vscode-server`，若不能访问外网可以在有网络的机器上下载离线安装包，然后上传到内网服务器。

## 下载vscode-server
Vscode远程连接服务器，需要保证本机Vscode和服务器上的`vscode-server`的 commit 版本一致，本机Vscode 的 commit id 可以打开**帮助->关于**界面查看到，例如：
```
版本: 1.77.3 (user setup)
提交: 704ed70d4fd1c6bd6342c436f1ede30d1cff4710  --这里是 commit id
日期: 2023-04-12T09:16:02.548Z
Electron: 19.1.11
Chromium: 102.0.5005.196
Node.js: 16.14.2
V8: 10.2.154.26-electron.0
OS: Windows_NT x64 10.0.22621
沙盒化: No
```

然后下载对应commit id 的 vscode-server 安装包，下载地址如下：
```bash
# 注意commit: 后面的id要对应上
https://update.code.visualstudio.com/commit:704ed70d4fd1c6bd6342c436f1ede30d1cff4710/server-linux-x64/stable
```

## 上传vscode-server并解压安装
vscode-server 安装在用户目录下，如`/home/xxx/.vscode-server`，下面是安装过程：
```bash
# commit id 自行替换
cd ~
wget https://update.code.visualstudio.com/commit:704ed70d4fd1c6bd6342c436f1ede30d1cff4710/server-linux-x64/stable -O server-linux-x64.tar.gz
mkdir -p ~/.vscode-server/bin/704ed70d4fd1c6bd6342c436f1ede30d1cff4710
tar -zxvf vscode-server-linux-x64.tar.gz -C ~/.vscode-server/bin/704ed70d4fd1c6bd6342c436f1ede30d1cff4710 --strip 1
```

## 免密登录
在 windows 上使用 `ssh-keygen -t rsa` 一路 enter 后生成密钥对，生成的密钥文件路径在 windows 用户目录的 `.ssh` 文件夹下，例如 `C:\Users\Administrator\.ssh`，将**公钥文件** id_rsa.pub 的内容复制添加到服务器的 `~/.ssh/authorized_keys` 文件中，最后在本机 vscode ssh 远程配置文件中配置**私钥文件**路径，如:
```
Host develop
  HostName 192.168.1.88
  User root
  Port 22
  IdentityFile "C:\Users\Administrator\.ssh\id_rsa"
```

## 参考
- [How can I install vscode-server in linux offline](https://stackoverflow.com/questions/56671520/how-can-i-install-vscode-server-in-linux-offline)