## 切换成 bash
> 针对 ubuntu 系统。
1. 查看当前使用的 sh
    ```
    ls -al /bin/sh

    lrwxrwxrwx 1 root root 4 Jul 20 14:55 /bin/sh -> dash
    ```

2. 使用bash作为默认shell
    ```
    sudo dpkg-reconfigure dash
    ```
    在弹出的对话框中选择 `NO`。

再次查看默认shell。

## 安装fish
```
sudo apt install fish
```
安装后输入 fish 进入，由于 Fish 的语法与 Bash 有很大差异，Bash 脚本一般不兼容，因此不建议把它作为默认shell，而是每次手动启动。
如果是本机使用（非ssh远程登陆），可以使用 `help` 命令，它会自动打开浏览器，显示在线文档。

## 安装oh my fish
安装步骤参考[官方github](https://github.com/oh-my-fish/oh-my-fish)。

下载安装完毕后，可以选择一个主题，例如：`omf theme bobthefish`，我个人比较喜欢该主题，该主题需要字体支持，推荐[Caskaydia Cove NF](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/CascadiaCode/Light/complete) 字体。

## 配置 fish
- 支持bash的`sudo !!`

    fish 的函数定义统一放在 `~/.config/fish/functions` 目录下，在该目录下创建一个名为 `bind_bang.fish` 文件并定义一个`bind_bang`函数：
    ```
    function bind_bang
        switch (commandline -t)
            case "!"
                commandline -t -- $history[1]
                commandline -f repaint
            case "*"
                commandline -i !
        end
    end
    ```
    然后在 `~/.config/fish/config.fish`文件中(没有则创建该文件)输入如下内容：
    ```
    # 绑定 !
    bind ! bind_bang
    ```
    退出fish后重进，就可以使用bash中的 `sudo !!`了。

    若fish版本较低，还可以使用第二种方法，直接在 `~/.config/fish/config.fish` 定义一个名为 `sudo` 的函数：
    ```
    function sudo
        if test "$argv" == !!
            eval command sudo $history[1]
        else
            command sudo $argv
        end
    end
    ```

- alias

    可以在 `~/.config/fish/config.fish` 定义命令别名，我的常用命令别名如下：
    ```
    #alias
    # 需要安装rlwrap
    alias telnet="rlwrap -I telnet"
    alias d='date +"%Y-%m-%d %H:%M:%S"'
    ```

- 设置环境变量参数

    同上，可以在配置文件种设置环境变量参数，如下：
    ```
    #exports
    export HISTTIMEFORMAT='%F %T '
    ```