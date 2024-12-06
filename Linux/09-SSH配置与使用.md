## 主机指纹

当第一次用 SSH 连接一个服务器主机时，ssh 客户端会有如下提示：

```
$ ssh 192.168.100.123
The authenticity of host '192.168.100.123 (192.168.100.123)' can't be established.
ECDSA key fingerprint is SHA256:QUfCwW6Br5EwwESsulN2TEidBoDNca888RNflZG++bI.
Are you sure you want to continue connecting (yes/no)? yes
```

上面提示主机 `192.168.100.123` 的真实性未确认，当输入 yes 后，会把服务器主机的 host 公钥记录在客户端机的 `.ssh/known_hosts` 中，形如：

```
192.168.100.123 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBF7Nq0of0/AW+Tphtm1ESs8stGAxxSfQvGYyk0VGPEshHOwU/A6Y4BBIBOjE7egqxkhapxf+BdNmH98DyYhQRps= root@DESKTOP-HP
```

进入服务器主机的 `/etc/ssh` 目录，可以看到一些带有 `host` 名称的公私钥对，例如：

```
ll /etc/ssh/ssh_host_*
-rw------- 1 root root  513 Mar 27 12:09 /etc/ssh/ssh_host_ecdsa_key
-rw-r--r-- 1 root root  182 Mar 27 12:09 /etc/ssh/ssh_host_ecdsa_key.pub
-rw------- 1 root root  411 Mar 27 12:09 /etc/ssh/ssh_host_ed25519_key
-rw-r--r-- 1 root root  102 Mar 27 12:09 /etc/ssh/ssh_host_ed25519_key.pub
-rw------- 1 root root 2.6K Mar 27 12:09 /etc/ssh/ssh_host_rsa_key
-rw-r--r-- 1 root root  574 Mar 27 12:09 /etc/ssh/ssh_host_rsa_key.pub
```

可以在客户机上使用下面命令计算服务器主机指纹：

```
ssh-keyscan -t ECDSA -p 22 192.168.100.123 2>/dev/null | ssh-keygen -E sha256 -lf -
```

详情参考：[验证远程主机 SSH 指纹](https://www.cnblogs.com/rongfengliang/p/10448225.html)。

## SSH 无密码登录

`ssh-keygen` 产生公钥与私钥对，`ssh-copy-id` 将指定的公钥复制到远程机器的 authorized_keys 文件中，ssh-copy-id 能让你有到远程机器的 `home`, `~./ssh` 和 `~/.ssh/authorized_keys`的权利。

具体命令如下：

```bash
# -i：指定公钥文件
ssh-copy-id -i .ssh/id_rsa.pub  用户名字@192.168.x.xxx

# 若没有 ssh-copy-id 命令，可以使用该方法
cat ~/.ssh/id_rsa.pub | ssh 用户名字@192.168.x.xxx "mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

**注意：** 要确保 `.ssh/id_rsa`的权限为 `600`, `.ssh/id_rsa.pub`的权限为 `644`，否则会出现`bad permissions: ignore key: /root/.ssh/id_rsa` 的报错。

## SSH 远程登录 WSL

在 wsl 上修改 sshd 配置文件 `/etc/ssh/sshd_config`，修改以下配置:

```
# 防止与windows主机上的sshd 监听端口22发生冲突
Port 2222
ListenAddress 0.0.0.0

# 默认也no，这会导致无法将公钥文件写入到 wsl 的 ~/.ssh/authorized_keys 中
PasswordAuthentication yes
```

重启 wsl 的 sshd 服务后，在客户端机器上使用命令 `ssh -p2222 root@192.168.0.xxx`，然后输入用户密码后即可登录 wsl。

当然为了安全起见，最好先使用 ssh-copy-id 命令先把客户机的公钥文件复制到目标主机，即 wsl 上，然后设置回 `PasswordAuthentication no`，这样可以禁止 SSH 远程连接的 password 认证，转而使用密钥登录。

另外还需要注意 windows 防火墙设置。

## know_hosts ip 加密

修改 ssh 客户端配置 `/etc/ssh/ssh_config` 中的 `HashKnownHosts` 选项，将其设置为 `yes`，则 know_hosts 文件中主机的 IP 将会以 hash 的方式保存。

如果想要查询某个 ip 的主机是否存在，则可以使用下面的命令：

```bash
ssh-keygen -F <hostname_or_ip> -f ~/.ssh/known_hosts
```

参考: [SSH known_hosts 显示 IP 地址](https://zdyxry.github.io/2019/12/06/SSH-known_hosts-%E6%98%BE%E7%A4%BA-IP-%E5%9C%B0%E5%9D%80/)。
