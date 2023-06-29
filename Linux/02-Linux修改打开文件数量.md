## 查看open file 数量
输入`ulimit -a` 可以查看当前系统的所有的资源限制，显示信息如下：
```
Maximum size of core files created                           (kB, -c) 0
Maximum size of a process’s data segment                     (kB, -d) unlimited
Maximum size of files created by the shell                   (kB, -f) unlimited
Maximum size that may be locked into memory                  (kB, -l) 64
Maximum resident set size                                    (kB, -m) unlimited
Maximum number of open file descriptors                          (-n) 1024
Maximum stack size                                           (kB, -s) 8192
Maximum amount of cpu time in seconds                   (seconds, -t) unlimited
Maximum number of processes available to a single user           (-u) 7823
Maximum amount of virtual memory available to the shell      (kB, -v) unlimited
```
其中，`Maximum number of open file descriptors` 表示了系统当前最大打开文件数量限制，一般默认为 **1024**，当时通常这个值过小，例如nginx服务器，很容易就会达到最大打开文件数量的限制。

## 临时修改
可以使用 `ulimit -n` 直接修改当前 shell 会话的 open file 数量，例如: `ulimit -n 10240`。但是退出shell后，设置就会清除。

## 永久修改
编辑 `cat /etc/security/limits.conf` 文件，在文件最后添加如下配置信息：
```
* soft nofile 65535
* hard nofile 65535
```
修改完后，需要**重启系统**才能生效。

## 修改运行中进程的open file
修改正在运行的进程的open file 数量，可以使用 `prlimit` 命令，先查询出进程的pid，再查下进程的 open file 数量。例如：
```
prlimit --pid=856
```

显示如下信息：
```
RESOURCE   DESCRIPTION                             SOFT      HARD UNITS
AS         address space limit                unlimited unlimited bytes
CORE       max core file size                         0 unlimited bytes
CPU        CPU time                           unlimited unlimited seconds
DATA       max data size                      unlimited unlimited bytes
FSIZE      max file size                      unlimited unlimited bytes
LOCKS      max number of file locks held      unlimited unlimited locks
MEMLOCK    max locked-in-memory address space     65536     65536 bytes
MSGQUEUE   max bytes in POSIX mqueues            819200    819200 bytes
NICE       max nice prio allowed to raise             0         0
NOFILE     max number of open files               65536     65536 files
NPROC      max number of processes                 7823      7823 processes
RSS        max resident set size              unlimited unlimited bytes
RTPRIO     max real-time priority                     0         0
RTTIME     timeout for real-time tasks        unlimited unlimited microsecs
SIGPENDING max number of pending signals           7823      7823 signals
STACK      max stack size                       8388608   8388608 bytes
```

其中，`NOFILE  max number of open files  65536 65536 files` 这一行标明了该进程的 soft 和 hard open file 数量。使用 `--nofile=10240:10240` 可以同时修改 soft 和 hard 两个 open file 数量，例如：
```
prlimit --pid=856 --nofile=10240:10240
```

更多参数可以查询帮助文档 `prlimit --help`。