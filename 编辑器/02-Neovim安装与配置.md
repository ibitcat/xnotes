系统环境为：**ubuntu 22.04 wsl**。

## 安装
安装方法参考[官网文档-installing](https://github.com/neovim/neovim/wiki/Installing-Neovim#Ubuntu)。

为了使用最新版的 Neovim，我注册了 unstable 的 PPA 源：https://launchpad.net/~neovim-ppa/+archive/ubuntu/unstable 。注册方式如下：
```
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt update
```

源更新完毕后，使用 `sudo apt install neovim` 安装，运行命令 `nvim --version` 检查 neovim 版本。

## 配置
配置文件使用该项目：[nvimdots](https://github.com/ayamir/nvimdots)。

在使用该配置前需要安装好相关依赖，包括：
- python相关
```shell
sudo apt install python3
sudo ln -s /usr/bin/python3 /usr/bin/python
#检测是否已经安装了 python venv
python -m venv "venv"
sudo apt install python3.10-venv
#pip
sudo apt install python3-pip
pip install neovim --user
pip3 install pynvim
```

- sqlite3
```
sudo apt install sqlite3
sudo ln -s /usr/lib/x86_64-linux-gnu/libsqlite3.so.0 /usr/lib/x86_64-linux-gnu/libsqlite3.so
```

- ripgrep
```
sudo apt-get install ripgrep
```

- fd
```
sudo apt install fd-find
mkdir .local/bin
ln -s /usr/bin/fdfind .local/bin/fd
# 确保 .local/bin 在 PATH 中，一般 ubuntu 已经在 .profile 中设置了
```

- nodejs
```
wget https://nodejs.org/dist/v18.12.1/node-v18.12.1-linux-x64.tar.xz
xz -d node-v18.12.1-linux-x64.tar.xz
tar -xf node-v18.12.1-linux-x64.tar
mv node-v18.12.1-linux-x64 /usr/local/node
sudo ln -s /usr/local/node/bin/node /usr/local/bin/node
sudo ln -s /usr/local/node/bin/npm /usr/local/bin/npm
```

- efm-langserver
```
sudo apt install efm-langserver
```

- win32yank
```
#解决 wsl 复制报错问题
curl -sLo win32yank.zip https://github.com/equalsraf/win32yank/releases/download/v0.0.4/win32yank-x64.zip
unzip win32yank.zip
sudo chmod +x win32yank.exe
sudo mv win32yank.exe  /usr/local/bin/
```

## 解决报错的方法
1. 查看 neovim 日志

    neovim 日志路径在 `~/.local/state/nvim`，例如 python venv 未安装时，mason插件日志在 `~/.local/state/nvim/mason.log`。

2. 使用 `:checkhealth` 检查 neovim

3. 打印调试 lua 插件

    若某个插件有报错，可以通过打印的方式对其进行调试，例如：
    ```lua
    print("xxxxx")  -- lua 原生打印函数
    vim.pretty_print("xxxx")    -- neovim 打印函数
    print(vim.inspect(vim))     -- inspect 可以格式化table
    vim.notify(vim.inspect({a=1,b=2}),"info")   -- 使用nvim-notify插件弹窗显示信息
    ```
    **注意**：若使用打印函数无法看到输出，可以使用命令 `:messages` 开启打印输出。

    另外，也可以直接使用命令执行打印，如：`:lua vim.notify(vim.inspect({a=1,b=2}),"info")`，若弹窗过快，可以使用`:Notifications`显示弹窗历史记录。
