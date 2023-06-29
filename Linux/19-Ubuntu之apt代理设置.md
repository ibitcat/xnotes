## 使用场景
内网机器 A (假设ip为192.168.1.200)有访问外网 apt源的权限，而其他内网机器则不能访问apt源，但是可以访问内网机器 A，因此可以在 A 上创建一个代理，让内网的其他机器通过代理来访问 apt 源。

## 使用clash开启直连代理
首先要在有外网apt访问权限的机器上用 clash 开启直连代理。

在 Clash 的 [release页面](https://github.com/Dreamacro/clash/releases)下载相应的版本，Ubuntu选择`clash-linux-amd64-v1.xx.x.gz`，解压后添加可执行权限，另外 Clash 运行需要 `Country.mmdb` 文件，可以在 [github](https://github.com/Dreamacro/maxmind-geoip/releases) 上手动下载。
> Country.mmdb 文件利用 GeoIP2 服务能识别互联网用户的地点位置，以供规则分流时使用。

```bash
wget https://github.com/Dreamacro/clash/releases/download/v1.15.1/clash-linux-amd64-v1.15.1.gz
gunzip clash-linux-amd64-v1.15.1.gz
mv clash-linux-amd64-v1.15.1 clash-linux-amd64
sudo chmod +x clash-linux-amd64
mkdir -p ~/.config/clash
```

接下来要创建 clash 的配置文件 `config.yaml`，clash 的配置文件夹在当前用户的 `~/.config/clash`，如果是手动下载 `Country.mmdb` 文件，也需要把它移动到配置文件夹下，直连代理的配置如下：
```yaml
mixed-port: 7890
allow-lan: true
log-level: info
mode: direct
```
然后运行 clash，运行成功会有以下输入：
```
INFO[0000] Mixed(http+socks) proxy listening at: [::]:7890
```

## 设置apt代理
在内网其他机器设置 apt 代理有三种方式：
1. 设置http_proxy环境变量
    ```bash
    # 这种设置只在当前会话shell中有效，退出终端后需要重新设置环境变量
    export http_proxy=http://192.168.1.200:7890
    apt-get update
    ```

2. apt命令临时使用代理
    ```bash
    # 使用-o参数
    sudo apt-get -o Acquire::http::proxy="http://192.168.1.200:7890/" update
    ```

3. 修改apt包管理器配置文件

    修改 `/etc/apt/apt.conf` 文件(没有就新增)，新增如下内容：
    ```
    Acquire::http::proxy "http://192.168.1.200:7890/";
    Acquire::ftp::proxy "ftp://192.168.1.200:7890/";
    Acquire::https::proxy "https://192.168.1.200:7890/";
    ```

## 参考
- [在 Linux 中使用 Clash](https://blog.iswiftai.com/posts/clash-linux/)
- [如何在 Linux 上优雅的使用 Clash？](https://blog.zzsqwq.cn/posts/how-to-use-clash-on-linux/)
- [Ubuntu apt 包管理器代理设置](https://www.expoli.tech/articles/2022/12/16/config-proxy-for-apt-package-manager)