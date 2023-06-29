首先要先熟悉以下 WSL 的[官方文档](https://learn.microsoft.com/zh-cn/windows/wsl/)，文档有很多 WLS1 和 WSL2 的相关信息。

## WSL导入和导出
导出前使用 `wsl -l -v` 列出当前已安装的wsl信息，包括Linux分发名、运行状态、wsl版本。
- 导出wsl
    ```
    wsl --export Ubuntu Ubuntu22.04.tar
    如果是 wsl2，也可以导出为 vhdx 文件
    wsl --export Ubuntu Ubuntu22.04.vhdx --vhd
    ```

- 导入wsl1
    ```
    wsl --import Ubuntu 'D:\Program Files\wsl\Ubuntu' 'D:\Program Files\wsl\Ubuntu22.04.tar' --version 1
    ```

- 导入wsl2
    ```
    wsl --import Ubuntu 'D:\Program Files\wsl\Ubuntu' 'D:\Program Files\wsl\Ubuntu22.04.tar' --version 2
    或者
    wsl --import Ubuntu 'D:\Program Files\wsl\Ubuntu' 'D:\Program Files\wsl\Ubuntu22.04.vhdx' --vhd --version 2
    ```

导入后，默认的用户是 `root`，可以通过修改注册表 `计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss` 下项的 `DefaultUid` 的值为十进制的 1000 即可。

## WSL1 tail -f 失效解决
在较旧版本的 win10 上安装的 wsl1，`tail -f` 无法监听问题的变动，原因是 Linux 是通过inotify来获取文件变动的，但 wsl 的 inotify 有bug（最新的系统好像已修复，未验证），可以使用 `tail -f ---disable-inotify` 来解决。为了方便使用，可创建一个命令别名 `alias tailf='tail -f ---disable-inotify'`。

>上述情况只发生在wsl中tail -f windows上的文件时才会出现，且无法使用grep；若tail -f wsl系统内的文件则正常。

## Landscape 报权限错误
在 windows11 系统下启动 Ubuntu 22.04，会在第一次启动 VM 时报以下错误：
```
/etc/update-motd.d/50-landscape-sysinfo: 17: cannot create /var/lib/landscape/landscape-sysinfo.cache: Permission denied
```

解决方法如下：
```
sudo apt remove landscape-common
sudo apt autoremove
rm ~/.motd_shown
```
参考[stackoverflow该问题](https://askubuntu.com/questions/1414483/landscape-sysinfo-cache-permission-denied-when-i-start-ubuntu-22-04-in-wsl/1414536#1414536?newreg=5cca9ddee1a74c06838ec0b4e733aa07)。
出现的原因在该问题的回答中有解释，大致是因为 WSL 的 Ubuntu 中没有 Systemd 而引起的报错。

## 安装 mlocate 卡住
Wsl2 的 Ubuntu 系统默认没有安装 mlocate，因此 `locate` 命令无法使用，可以使用 `sudo apt install mlocate` 来安装。

但是在安装完毕后的数据库初始化步骤，安装进度会被卡住，显示如下信息：
```
Initializing plocate database; this may take some time...
```
卡住的原因参见这个[issue](https://github.com/microsoft/WSL/discussions/5967)，即 wsl2 与宿主主机之间的 IO 慢，造成 mlocate 数据库初始化也很慢。

以下是解决方案：在 `/etc/updatedb.conf` 的 `PRUNEPATHS` 字段添加 `/mnt`，即 mlocate 在初始化时排除 `/mnt` 路径。

## 桥接网络并固定IP
1.0 以上版本的 wsl2 已经解决了网络问题，可以参考 [WSL2使用桥接网络，并指定IP](https://www.cnblogs.com/lic0914/p/17003251.html) 进行配置，亲测可行。

## WSL2开启systemd
在 wsl2 的 `/etc/wsl.conf` 文件中(没有就新建一个)添加以下配置：
```
[boot]
systemd = true
```
注意，systemd 需要 wsl2，且wsl版本要为 0.67.6 或以上。