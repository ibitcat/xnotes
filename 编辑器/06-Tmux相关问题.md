## 安装 tmux 配置

使用 [Oh my tmux](https://github.com/gpakosz/.tmux) 开源配置，安装步骤参考官方 readme。

若检查配置无误后，tmux 配置还不生效，则执行 `tmux source ~/.tmux.conf` 强制加载配置。

## 自定义配置

编辑 `.tmux.conf.local` 本地配置文件，添加以下内容：

```bash
# 设置历史记录长度
set -g history-limit 10000

# 开启鼠标操作
set -g mouse on

# 复制模式使用 vi 风格的键绑定
set -g mode-keys vi

# 状态栏powerline 显示设置
tmux_conf_theme_left_separator_main='\uE0B0'
tmux_conf_theme_left_separator_sub='\uE0B1'
tmux_conf_theme_right_separator_main='\uE0B2'
tmux_conf_theme_right_separator_sub='\uE0B3'
```

状态栏的 powerline 显示需要终端的字体支持，在 [nerdfonts](https://www.nerdfonts.com/font-downloads) 下载 **CaskaydiaMono Nerd Font** 字体并安装。

## 脚本自动化

以下是一个分屏并执行命令的自动化脚本，它的具体功能如下：

1. 创建一个 session；
2. 在这个 session 中分别创建三个名为 center、game、login 的窗口；
3. 将每个窗口垂直分割为两个窗格；
4. 在每个窗口的第一个窗格执行脚本；
5. 让每个窗口的第二个窗格作为活动窗格；

```bash
echo "create center window ..."
# 创建一个名为 s1 的session(会话)，并自动创建第一个window(窗口)
# -P 参数返回会话、窗口、窗格的信息
tmux new -s s1 -n center -d -P
tmux split-window -h -t s1:1 -d -P

nodes=("center" "game" "login")
nodelen=${#nodes[@]}
for ((i=1;i<nodelen;i++)); do
    echo "create ${nodes[$i]} window ..."
    tmux new-window -t s1 -n ${nodes[$i]} -d -P
    tmux split-window -h -t s1:$((i+1)) -d -P
    sleep 0.3
    echo ""
done

#tmux send-keys -t s1 'sh server.sh start center' Enter
for ((i=0;i<nodelen;i++)); do
    echo "start ${nodes[$i]} ..."
    stopcmd="sh server.sh stop ${nodes[$i]}"
    echo $stopcmd
    tmux send-keys -t s1:$((i+1)).1 "$stopcmd" Enter

    startcmd="sh server.sh start ${nodes[$i]}; tail -f /data/log/server_${nodes[$i]}/skynet-$(date +"%Y-%m-%d").log"
    echo $startcmd
    tmux send-keys -t s1:$((i+1)).1 "$startcmd" Enter
    sleep 0.3

    tmux select-pane -t s1:$((i+1)).2
    echo ""
done
```

在 tmux 中，每个窗格的标识符格式为：<session_name>:<window>.<pane> 例如 dev:1.1 表示会话 dev 的第 1 个窗口的第 1 个窗格。
可以用 `tmux list-pane -a` 列出所在窗格。例如：

```
s1:1.1: [105x56] [history 0/10000, 2832 bytes] %0 (active)
s1:1.2: [104x56] [history 0/10000, 2052 bytes] %1
s1:2.1: [40x24] [history 0/10000, 968 bytes] %2 (active)
s1:2.2: [39x24] [history 0/10000, 963 bytes] %3
s1:3.1: [40x24] [history 0/10000, 968 bytes] %4 (active)
s1:3.2: [39x24] [history 0/10000, 963 bytes] %5
```

也可以列出指定会话的窗格列表，命令如下：

```bash
tmux list-panes -s -t $SESSION -F "#{session_name}:#{window_index}.#{pane_index}"
```

输出示例：

```
s1:1.1
s1:1.2
s1:2.1
s1:2.2
s1:3.1
s1:3.2
```
