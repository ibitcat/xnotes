系统环境为：win11 最新版的 **ubuntu 22.04 wsl2**。

在 WLS2 安装使用 docker 有以下两种方式：
- 安装Docker Desktop for Windows（设置 wsl2 访问）
- 在 wsl2 系统内原生安装 docker

## Docker Desktop for Windows
安装文档可以参考微软官方文档：[WSL 2 上的 Docker 远程容器入门](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-containers)。
根据文档一路安装配置后即可在 wsl2 内使用 docker。

需要注意的是，在 “Settings” -> “Docker Engine” 中修改配置时要确保配置是正确的，若配置有问题会导致 Docker Desktop 崩溃，就算点击了恢复默认配置也要卡很久，例如在配置里面加入了一个已经失效的镜像源。

一个解救的办法是：在用户目录 `C:\Users\<用户名>\.docker` 创建 `daemon.json` 配置文件，例如：
```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://mirror.ccs.tencentyun.com"
  ]
}
```
然后重启 Docker Desktop。


## WSL2 原生安装
这种方法需要一些先决条件，即系统版本要比较新，因为最新的 wsl2 已经原生支持了 `service` 命令。在 linux 上安装 docker 可参见 docker 官方的安装文档 [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

推荐使用官方的一键安装脚本，命令如下：
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --dry-run
sudo service docker start
# 测试docker运行
docker run hello-world
```

这里有一点需要注意，若卸载已安装的 Docker Desktop for Windows 后，需要删除 wsl2 中用户目录的 `.docker` 文件夹，否则在使用 `docker-compose` 启动时会报以下错误：
```
error getting credentials - err: docker-credential-desktop.exe resolves to executable in current directory (./docker-credential-desktop.exe), out: ``
```

## 遇到的问题
- 使用 `docker network prune` 删除网络后再次启动容器时出现以下报错
```
Error response from daemon: network f29b91c9793d6b5a6d363b7602155e358ad859d3749638b681ffbf09707ce8df not found
```
可以使用 `docker-compose up -d --force-recreate` 强制重新创建网络。

- 在容器中安装telnet
```
# 进入容器sh
docker exec -it e08f889ef3b4 /bin/sh
# 替换源镜像(不然奇慢无比)
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
# 安装 busybox-extras
apk add busybox-extras
busybox-extras telnet localhost 3000
```
另外，在容器内检测端口是否开启，还可以使用 `netcat` 来检查。命令如下
```
nc -zv 127.0.0.1 3000
```

- 禁止 docker 自动启动
```
sudo systemctl disable docker
sudo systemctl disable docker.socker
```
主要注意 `docker.socket` 这个服务，在停止 `docker` 服务时会提示以下信息：
```
Warning: Stopping docker.service, but it can still be activated by: docker.socket
```
这个 `docker.socker` 服务是为了自动唤醒 docker 的，例如在 stop 了 docker 服务后，又运行了一个 docker 命令，此时 docker 服务将会被自动唤醒。
>需要在 wsl2 中启用 systemd.