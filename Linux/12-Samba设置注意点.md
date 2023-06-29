## 坑爹的 selinux

Linux 上的 selinux 非常之蛋疼，如果设置不好会产生很多权限的问题。通过 `getenforce` 可以查看 selinux 的状态，例如：
```
bitcat@surface-book2:~$ sudo getenforce
Enforcing       # selinux 开启状态
Permissive      # selinux 关闭状态
```

可以使用命令 `setenforce` 临时改变 selinux 的状态。
```
setenforce 1 # 开启
setenforce 0 # 关闭
```

也可以永久关闭 selinux，编辑 `/etc/selinux/config` 配置文件，将 `ELINUX=enforcing` 改为 `SELINUX=disabled` 即可。

## Samba 能在windows 显示文件夹但无法打开
- 方法1，简单粗暴，直接关闭 selinux 一劳永逸。
- 方法2，设置 selinux
    1. 修改目录权限
    ```bash
    # 先查看 xxx 的selinux权限, -Z 选项可以显示文件的安全上下文信息
    ls -lZ /data
    # drwxr-xr-x. 3 root root unconfined_u:object_r:samba_share_t:s0  4096 9月   1 22:55 xxx
    # 其中 unconfined_u:object_r:samba_share_t:s0 就是文件的 security context

    chcon -R -t samba_share_t /data/xxx
    ```

    2. 设置 selinux 开关
    ```bash
    # 查看 selinux samba 开关
    getsebool -a |grep samba
    # 输出如下
    samba_create_home_dirs --> off
    samba_domain_controller --> off
    samba_enable_home_dirs --> off
    samba_export_all_ro --> off
    samba_export_all_rw --> off
    samba_load_libgfapi --> off
    samba_portmapper --> off
    samba_run_unconfined --> off
    samba_share_fusefs --> off
    samba_share_nfs --> off
    sanlock_use_samba --> off
    tmpreaper_use_samba --> off
    use_samba_home_dirs --> off
    virt_use_samba --> off

    # 开启 samba_export_all_rw
    setsebool samba_export_all_rw on -P
    ```

## Git pull 报 "Unlink of file" 错误
参考 stackoverflow 的[这个问题](https://stackoverflow.com/questions/62450100/git-with-samba-on-ubuntu-unlink-of-file)。回答中给出了微软官方的解释和解决方法，详见 [SMB2 Client Redirector Caches Explained](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-7/ff686200(v=ws.10)#details)。

为了方便，可以将下面的代码保存为 `.reg` 文件(注意**要用windows记事本编辑**)，双击运行即可。
```
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters]
"DirectoryCacheLifetime"=dword:00000000
"FileInfoCacheLifetime"=dword:00000000
"FileNotFoundCacheLifetime"=dword:00000000
```

>为什么要用 windows 记事本编辑？因为记事本的默认编码是ANSI，在简体中文Windows操作系统中，ANSI 编码代表 GBK 编码。当然也可以使用其他编辑器，只要选择正确编码格式即可。