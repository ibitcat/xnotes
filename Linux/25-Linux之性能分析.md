## CPU 性能分析

### 查找高 cpu 进程

使用 top 类工具查找到 cpu 占比高的进程：

-   `P(大写)`，按 cpu 排序
-   `M(大写)`，按 内存排序

或者使用 `ps` 命令：

```bash
ps -aux --sort=-%cpu | head -5
ps -aux --sort=-%mem | head -5
```

查找到进程后，再进一步查看线程的 cpu 使用情况：

```bash
top -p <PID> -H
#或者
htop -p <PID> #然后按大写H
```

补充 top 各个指标的意义：

-   us：用户态
-   sy：内核态（含系统调用、内核函数）
-   wa：I/O 等待
-   hi/si：硬/软中断
-   st：被虚拟化抢占的时间

### 查看系统调用情况

使用 `strace` 来查看进程的系统调用：

```
sudo apt install strace
strace -c -f -e trace=read,write,futex -p <PID>
```

-   `-c`,统计系统调用耗时和次数
-   `-f`,跟踪子进程 / 线程
-   `-p`,附加到已有进程
-   `-e`,跟踪过滤(包括系统调用(read,write,futex)、信号、网络调用(socket,send,recv))

### 查找耗时的函数调用

使用 `perf` 来查看热点函数：

```bash
# 安装 perf 工具
sudo apt install linux-tools-generic
```

**注意：** wsl2 上安装 perf 会比较麻烦，因为 perf 工具与内核版本强绑定，因此需要自己编译：

1. 先安装上面的包，然后运行 `perf -v`

```bash
# 会有以下输出
WARNING: perf not found for kernel 6.6.87.2-microsoft

  You may need to install the following packages for this specific kernel:
    linux-tools-6.6.87.2-microsoft-standard-WSL2
    linux-cloud-tools-6.6.87.2-microsoft-standard-WSL2

  You may also want to install one of the following packages to keep up to date:
    linux-tools-standard-WSL2
    linux-cloud-tools-standard-WSL2
```

2. 下载指定版本的内核代码

```bash
git clone https://github.com/microsoft/WSL2-Linux-Kernel --depth 1 -b linux-msft-wsl-6.6.87.2
```

3. 编译 perf

```bash
# 安装所有依赖
sudo apt install -y \
  build-essential \
  flex bison \
  pkg-config \
  libelf-dev libdw-dev \
  libunwind-dev libzstd-dev \
  libcap-dev libpfm4-dev \
  libtraceevent-dev \
  libbabeltrace-dev libbabeltrace-ctf-dev \
  systemtap-sdt-dev \
  python3-dev libperl-dev \
  default-jdk

cd WSL2-Linux-Kernel/tools/perf
make -j8
sudo cp perf /usr/local/bin
```

4. 验证 perf

```bash
perf -v

# 如果编译整成，则输出如下
perf version 6.6.87.2.g427645e3db3a
```

### 实时监控

```bash
sudo perf top -p <PID>

# 只显得函数调用>=2%的
sudo perf top -p <PID>  --percent-limit=2

# 以每秒 99 的采样频率监控，并显示调用图(-g)
sudo perf top -g -p <PID> -F 99
```

### 离线采样

```bash
# 以 99HZ 的频率采样 60 s
sudo perf record -F 99 -p <PID> -g -- sleep 60

# 使用 report 查看
#用文本模式，更省内存
perf report --stdio -i perf.data

#关闭调用栈解析（最耗内存）
perf report --call-graph=none --max-stack 0 -i perf.data

# 火焰图
sudo perf record -F 99 -a -g -- sleep 60
sudo perf script > out.perf
git clone https://github.com/brendangregg/FlameGraph.git
cd FlameGraph

# 将堆叠样本折叠成单行
./stackcollapse-perf.pl ../out.perf > ../out.folded

# 导出svg
./flamegraph.pl ../out.folded > ../out.svg
```
