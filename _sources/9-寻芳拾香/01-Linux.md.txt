# Linux

## `grep` 搜索关键字

1. -A NUM  --after-context=NUM
2. -B NUM  --before-context=NUM
3. -C NUM  -NUM --context=NUM

## `top` 查看系统实时状况

### 示例

```yaml
top - 17:22:10 up 1 day,  6:38,  2 users,  load average: 2.64, 3.47, 3.30
Tasks: 315 total,   1 running, 314 sleeping,   0 stopped,   0 zombie
%Cpu(s):  6.7 us,  2.3 sy,  0.0 ni, 90.0 id,  0.9 wa,  0.0 hi,  0.2 si,  0.0 st
MiB Mem :  11862.6 total,    339.1 free,   8378.9 used,   3144.5 buff/cache
MiB Swap:  15624.0 total,  11226.0 free,   4398.0 used.   2314.6 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   2275 lighk     20   0 6369944 439684  39808 S  16.6   3.6  77:34.25 gnome-shell
  99353 lighk     20   0   10.9g   3.1g  81984 S   7.6  26.8  79:28.11 java
   2068 lighk     20   0 1435040 103936  64888 S   7.0   0.9  81:56.48 Xorg

```

### 启动时间和平均负载

```yaml
top - 17:22:10 up 1 day,  6:38,  2 users,  load average: 2.64, 3.47, 3.30
```

- 程序或窗口名，取决于展示模式
- 当前时间和开机时长
- 总用户数
- 系统过去 1,5,15 分钟的平均负载

### 任务和 CPU 状态

```yaml
Tasks: 315 total,   1 running, 314 sleeping,   0 stopped,   0 zombie
%Cpu(s):  6.7 us,  2.3 sy,  0.0 ni, 90.0 id,  0.9 wa,  0.0 hi,  0.2 si,  0.0 st
```

|标识 | 全称 | 说明 |
|---|--| --|
| `us` | `user`               | 没有设置 nice 值的用户进程运行时间|
| `sy` | `system`             | 内核进程运行时间|
| `ni` | `nice`               | 设置了 nice 值的用户进程运行时间|
| `id` | `idle`               | 内核空闲时间|
| `wa` | `IO-wait`            | IO 等待时间|
| `hi` | `hardware interrupts`| 硬件中断消耗时间|
| `si` | `software interrupts`| 软件中断消耗时间|
| `st` | `stolen`             | 虚拟化层夺走的时间|

### 内存使用

```yaml
MiB Mem :  11862.6 total,    339.1 free,   8378.9 used,   3144.5 buff/cache
MiB Swap:  15624.0 total,  11226.0 free,   4398.0 used.   2314.6 avail Mem 
```

### 进程信息

|标识  | 含义   |
|---   | ---    |
| PID  | 进程ID  |
| USER | 用户名  |
| PR   | 优先级  |
| NI   | Nice 值，用于优先级控制  |
| VIRT | 虚拟内存  |
| RES  | 当前使用的物理内存（non-swapped）  |
| SHR  | 共享内存，RES的子集  |
| S    | 进程状态  |  
| %CPU | CPU使用率  |
| %MEM | 内存使用率  |
| TIME+| 进程运行时间  |
| COMMAND| 命令名 |  

进程状态：

- D = uninterruptible sleep
- I = idle
- R = running
- S = sleeping
- T = stopped by job control signal
- t = stopped by debugger during trace
- Z = zombie

### 查看线程状态

```bash
top -p $PID -H
```

## journalctl 日志工具

journalctl 是一个用于查询 systemd-journald 服务收集的日志的工具。以下是一些常用的 journalctl 命令用法：

- 查看所有日志： journalctl
- 查看特定服务的日志： journalctl -u [服务名称]
- 查看某个时间段的日志： journalctl --since [时间] --until [时间]
- 查看特定级别的日志： journalctl -p [级别]
- 查看某个特定事件的日志： journalctl -k [事件ID]
- 查看系统引导时的日志： journalctl -b
- 查看最近n条日志： ournalctl -n [数字]
- 实时查看日志： journalctl -f

## ftp

``` shell
# 登录 
ftp $host

# 使用 utf-8
quote opts utf8 on

# 下载
get filename
# 上传
put filename
```

## yum 离线安装软件

在本机下载软件安装包，就可以去机器进行离线安装

```bash
# 先安装 工具
yum install yum-utils

# 下载安装包(以 libffi 为例)
yumdownloader libffi
```

## tcp

### TCP 连接

```bash
➜  ~ netstat -an | grep 1521
tcp        0      0 10.10.14.34:35032       10.10.14.5:1521         ESTABLISHED
tcp        0      0 10.10.14.34:34816       10.10.14.5:1521         ESTABLISHED
tcp        0      0 10.10.14.34:34814       10.10.14.5:1521         ESTABLISHED
tcp6       0      0 10.10.14.34:52734       10.10.14.5:1521         ESTABLISHED
```


### TCP 抓包

```bash
➜  ~ tcpdump -v -n -X -i any port 1521
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
13:39:29.848038 IP (tos 0x0, ttl 128, id 17761, offset 0, flags [DF], proto TCP (6), length 41)
    10.10.14.5.ncube-lm > 10.10.14.34.34816: Flags [.], cksum 0xc3a7 (correct), seq 2959199793:2959199794, ack 2324825286, win 8212, length 1
	0x0000:  4500 0029 4561 4000 8006 8533 0a0a 0e05  E..)Ea@....3....
	0x0010:  0a0a 0e22 05f1 8800 b061 ce31 8a92 04c6  ...".....a.1....
	0x0020:  5010 2014 c3a7 0000 0000 0000 0000 0000  P...............
	0x0030:  0000 0000 0000 0000 0000 0000 0000       ..............
13:39:29.848056 IP (tos 0x0, ttl 64, id 23648, offset 0, flags [DF], proto TCP (6), length 52)
    10.10.14.34.34816 > 10.10.14.5.ncube-lm: Flags [.], cksum 0x3061 (incorrect -> 0xae0d), ack 1, win 623, options [nop,nop,sack 1 {0:1}], length 0
	0x0000:  4500 0034 5c60 4000 4006 ae29 0a0a 0e22  E..4\`@.@..)..."
	0x0010:  0a0a 0e05 8800 05f1 8a92 04c6 b061 ce32  .............a.2
	0x0020:  8010 026f 3061 0000 0101 050a b061 ce31  ...o0a.......a.1
	0x0030:  b061 ce32 0000 0000 0000 0000 0000 0000  .a.2............
	0x0040:  0000 0000                                ....
```