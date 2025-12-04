skynet 的内存可以分为两个部分：

-   lua 内存
-   C 内存

它们统一由 jemalloc 进行分配和回收。

## lua 内存

它是指 lua 虚拟机在运行 lua 脚本时所使用的内存，这部分内存由 lua 虚拟机自己管理，lua 的垃圾回收机制会自动回收不再使用的内存。可以通过 `collectgarbage("count")` 来查看，单位是 KB，在 debug console 中使用 `mem` 命令可以列出各个服务当前的 lua 内存使用情况。

以下是一个 lua 内存列表的示例输出：

```
mem
:00000004       57.37 Kb (snlua cdummy)
:00000006       55.54 Kb (snlua datacenterd)
:00000007       62.66 Kb (snlua service_mgr)
:00000008       61.96 Kb (snlua service_provider)
:00000009       57.23 Kb (snlua service_cell ltls_holder)
:0000000a       257.08 Kb (snlua node/center/main)
:0000000b       60.46 Kb (snlua clusterd)
:0000000c       79.83 Kb (snlua service_cell sharetable)
:00000011       96.71 Kb (snlua debug_console 33000)
:00000012       64.27 Kb (snlua gate)
:00000013       81.36 Kb (snlua gate_http http)
:00000014       70.99 Kb (snlua gate_http http)
:00000015       87.10 Kb (snlua inotify)
:00000016       219.35 Kb (snlua app/db/main 1 10)
:00000017       219.35 Kb (snlua app/db/main 2 10)
```

## C 内存

它是指 skynet 框架本身以及 C 语言编写的模块所使用的内存，即使用了 `skynet_malloc` 分配的内存，它会在分配内存时在记录分配信息在内存头部，以统计不同服务中的内存分配情况，在 debug console 中使用 `cmem` 命令可以列出各个服务当前的 C 内存使用情况，单位是 byte。

**注意**：最后的 `:fffffffx` 形式的地址是指非 worker 线程的内存分配。

以下是一个 C 内存列表的示例输出：

```
cmem
:00000001       5152
:00000002       367836
:00000003       53646
:00000004       64304
:00000005       112
:00000006       15236
:00000007       73787
:00000008       19556
:00000009       119389
:0000000a       1568186
:0000000b       33075
:0000000c       625598
:0000000d       4600
:0000000e       4600
:00000010       9200
:00000011       82488
:00000012       41715
:00000013       33675
:00000014       0
:00000015       23001
:00000016       58162
:00000017       3288
:fffffffe       40            <- 它其实是 socket 线程分配的内存， see skynet_imp.h
:ffffffff       11120950      <- 它其实是 主线程分配的内存
block   31405                 <- 当前 jemalloc 分配的内存块数量
total   14327596              <- jemalloc 分配的总内存数量
```

## jemalloc 内存统计

在 debug console 中使用 `jmem` 命令可以查看 jemalloc 的内存使用统计信息，示例如下：

```
jmem
stats.active       22827008     21.77 Mb
stats.allocated    21644688     20.64 Mb
stats.mapped       50466816     48.13 Mb
stats.resident     28430336     27.11 Mb
stats.retained     18214912     17.37 Mb
```

以下是各字段的含义：

-   stats.active：当前活跃的内存总量，即 jemalloc 分配出去但未释放的内存总量。
-   stats.allocated：当前已分配的内存总量，即应用程序实际使用的内存总量。
-   stats.mapped：当前映射的内存总量，即 jemalloc 从操作系统申请的内存总量，换句话说，只要 jemalloc mmap 过但没 munmap 的内存，都在 mapped 中。
-   stats.resident：当前驻留的内存总量，即实际驻留在物理内存中的内存总量，即 top 中的 rss 值。
-   stats.retained：当前保留的内存总量，即 jemalloc 但未使用的内存总量，它只是虚拟地址空间，不占物理内存。

它们之间的关系可以表示为：

-   `active ≈ allocated + 内部碎片`，例如：申请 1000 字节，jemalloc 会分配一个 4kb 的页，那么 allocated = 1000 字节，active = 4kb，多出来的 3kb 就是内部碎片。
-   `resident = active + 外部碎片(dirty pages + metadata)`，其中 dirty pages 是指已经分配但未使用的内存页，metadata 是 jemalloc 用于管理内存的元数据开销。**这是进程真正占用的物理内存**。
-   `mapped = (active + retained + dirty + muzzy) ≥ resident`，mapped 是从操作系统申请的虚拟内存空间，resident 是实际驻留在物理内存中的内存总量，mapped 包括了未使用的内存页（未真正分配 RAM）。

它们的关系可以用下图表示：

```
┌─────────────────────────────── stats.mapped (6216 MB) ──────────────────────────────┐
│  虚拟地址空间                                                                       │
│  ┌────────────────────────── stats.resident (6140 MB) ────────────────────────────┐ │
│  │  物理内存占用                                                                  │ │
│  │  ┌──────────────────── stats.active (5984 MB) ───────────────────────┐         │ │
│  │  │  激活的内存页                                                     │         │ │
│  │  │  ┌───────────── stats.allocated (5601 MB) ──────────────┐         │         │ │
│  │  │  │  应用实际使用                                        │         │         │ │
│  │  │  │                                                      │         │         │ │
│  │  │  │  [应用数据]                                          │         │         │ │
│  │  │  │                                                      │         │         │ │
│  │  │  └──────────────────────────────────────────────────────┘         │         │ │
│  │  │  [内部碎片: 383 MB]                                               │         │ │
│  │  └───────────────────────────────────────────────────────────────────┘         │ │
│  │  [Dirty Pages + Metadata: 156 MB]                                              │ │
│  └────────────────────────────────────────────────────────────────────────────────┘ │
│  [未使用的虚拟地址空间: 76 MB]                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

内存碎片率计算：

-   内部碎片率 = (stats.active - stats.allocated) / stats.allocated，基准值为 10%。
-   外部碎片率 = (stats.resident - stats.active) / stats.active

## jemalloc 内存分配机制

-   jemalloc 使用多个 arena 避免锁竞争
-   每个 arena 管理多个 extent (内存区域)
-   extent 包含多个 page，page 包含多个对象
-   释放对象后，page 可能变成 dirty/muzzy 状态，但不立即归还系统

**Decay 机制**：对象释放 → dirty page → (dirty_decay_ms) → muzzy page → (muzzy_decay_ms) → 归还系统。

-   `dirty page`：最近释放，内容未清零，可快速重用
-   `muzzy page`：较久未用，内容已失效，但仍在进程地址空间
-   `decay 时间`：控制从释放到归还系统的延迟

## jemalloc 调优

jemalloc 可以通过设置 MALLOC_CONF 环境变量来进行调优，常用的参数有：

```bash
#注意：如果编译时启用了 --with-jemalloc-prefix=je_，则需要使用 JE_MALLOC_CONF 代替 MALLOC_CONF
export MALLOC_CONF="background_thread:true,dirty_decay_ms:1000,muzzy_decay_ms:1000"
```

-   `background_thread:true`：启用后台线程来异步回收内存，可以减少主线程的内存回收开销。
-   `dirty_decay_ms:1000`：设置脏页的衰减时间为 1000 毫秒，表示 jemalloc 会在后台线程中每隔 1 秒钟回收一次脏页。
-   `muzzy_decay_ms:1000`：设置 muzzy 页的衰减时间为 1000 毫秒，表示 jemalloc 会在后台线程中每隔 1 秒钟回收一次 muzzy 页。

## 内存碎片

在 skynet 中如果出现了内存碎片，可以通过以下方式手动触发 jemalloc 的内存回收：

```lua
local m = require("skynet.memory")
local narenas = m.mallctl("arenas.narenas")
for i=0,narenas-1 do
    m.mallctl(string.format("arena.%s.decay",i))
    m.mallctl(string.format("arena.%s.purge",i))
end
```

这个命令会遍历所有的 jemalloc arena，并调用 decay 和 purge 方法来回收内存碎片。

## THP 与 jemalloc 冲突

在使用 jemalloc 的同时启用 Linux 的透明大页（THP，Transparent Huge Pages）功能，可能会导致内存使用效率降低，甚至出现内存碎片过多的问题。这是因为 THP 会将多个小的内存页合并成一个大的内存页，而 jemalloc 则倾向于分配较小的内存块，这两者之间的冲突可能会导致内存分配和回收效率降低。

可以通过以下命令来检查、修改系统的 THP 状态：

```bash
# 查看当前系统 THP 使用情况
grep -i hugepages /proc/meminfo

# 查看 THP 是否启用
cat /sys/kernel/mm/transparent_hugepage/enabled
# [always] madvise never

# 查看碎片整理状态
cat /sys/kernel/mm/transparent_hugepage/defrag
# [always] defer defer+madvise madvise never

# 查看当前进程的 THP 使用情况
sudo cat /proc/<PID>/smaps | grep AnonHugePages

# 修改 THP 设置为禁用
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
#或按需使用
echo madvise | sudo tee /sys/kernel/mm/transparent_hugepage/enabled

# 修改碎片整理设置
echo madvise | sudo tee /sys/kernel/mm/transparent_hugepage/defrag
```
