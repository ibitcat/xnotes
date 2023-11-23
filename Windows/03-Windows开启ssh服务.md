系统版本：Win11 23H2

## 可选功能安装 Open SSH
打开设置页面，进入`应用➡️可选功能➡️添加可选功能`，输入 ssh 搜索，把 ssh 客户端和服务器都下载安装上，这部分下载过程有点慢。

安装完成后，先测试能否启动 sshd，用管理员权限打开 CMD，输入以下内容：
```bash
net start sshd
```

可以通过以下命令查看 sshd 进程是否启动:
```bash
netstat -ano |findstr /rc:"\<22\>"
```
ps：`/rc:"\<22\>"` 表示全字匹配，`/r` 表示正则匹配模式，`/c:"xxx"` 表示完整匹配。

## 修改端口号
端口 22 不太安全，可以改成其他的，例如: 2222，windows 下 ssh 相关的配置路径在 `C:\ProgramData\ssh`，修改 `sshd_config`,
注意：这里要用管理员模式打开这个文件，操作步骤：`开始🪟➡️ 搜索记事本➡️右侧选择以管理员身份运行`，然后打开配置文件，取消注释`Port 2222`。

## 设置防火墙
操作步骤：`控制面板➡️系统和安全➡️Windows Defender防火墙➡️高级设置➡️入站规则➡️新建规则`，然后为你的 sshd 监听端口添加新的入站规则。

## 以服务的方式开机启动
按`Win🪟+R`，输入 `services.msc` 打开服务管理界面，找到 `OpenSSH SSH Server`，将它的属性修改为**自动**，这样每次开机就会自动开启 sshd 服务。

## Linux 远程执行 Windows 命令
在 linix ssh 输入 `ssh 用户名@windows Ip 地址 -p 端口(如2222)`，输入密码后即可连接到windows，此时便可以执行诸如 `ipconfig`，`shutdown -r -t 5`(5s后重启) 等命令。
