> 监测服务器性能的基本命令

通过 CPU、内存、I/O 这三方面，回顾 Linux 系统下，服务器性能监测命令以及问题排查思路。

### CPU

#####  查看 CPU 软硬件信息

```shell
$ cat /proc/cpuinfo
processor       : 系统核编号，从 0 开始排序
vendor_id       : 制造商
cpu family      : 产品系列号
model           : 产品系列中的哪一代，代号
model name      : CPU 名称和编号，标称主频
stepping        : 更新版本
microcode       : 0x1d
cpu MHz         : 实际使用主频
cache size      : 二级缓存大小
physical id     : 单个 CPU 标号
siblings        : 单个 CPU 逻辑物理核数
core id         : 当前物理核在其所处 CPU 中的编号，不一定连续
cpu cores       : 该逻辑核所处 CPU 的物理核数
apicid          : 区分不同逻辑核的编号，每个逻辑核编号必然不同
initial apicid  : 初始化逻辑核编号
fpu             : 是否具有浮点运算单元（Floating Point Unit）
fpu_exception   : 是否支持浮点计算异常
cpuid level     : 执行 cpuid 指令前，eax 寄存器中的值，根据不同的值 cpuid 指令会返回不同的内容
wp              : 表明当前CPU是否在内核态支持对用户空间的写保护（Write Protection）
flags           : 当前CPU支持的功能
bogomips        : 在系统内核启动时粗略测算的CPU速度（Million Instructions Per Second）
clflush size    : 每次刷新缓存的大小单位
cache_alignment : 缓存地址对齐单位
address sizes   : 可访问地址空间位数
power management: 对能源管理的支持
```

常用过滤查找命令
```shell
# 查询系统CPU的个数
$ cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
2

# 查询系统具有多少个逻辑核
$ cat /proc/cpuinfo | grep "processor" | wc -l
8

# 查询系统CPU的物理核数
$ cat /proc/cpuinfo | grep "cpu cores" | uniq
cpu cores : 4

# 查询系统CPU是否启用超线程; 输出二者一致，没有启用超线程，否则被启用
$ cat /proc/cpuinfo | grep -e "cpu cores"  -e "siblings" | sort | uniq
cpu cores : 4
siblings  : 4
```



##### 查看进程运行信息

```shell
$ top

# -d：设置刷新间隔秒数
# -n：设置更新次数，到达指定次数后自动退出 top 视图
# -p：获取指定端口的进程信息

# 当前时间，系统运行天数；当前系统在线用户数；负载 1 分钟、5 分钟、15 分钟
top - 10:07:34 up 262 days, 16:41,  1 user,  load average: 0.02, 0.04, 0.05

# 共运行进程数，2 个正在运行，829 在休眠状态；0 个停止状态； 0 个僵尸进程
Tasks: 831 total,   2 running, 829 sleeping,   0 stopped,   0 zombie

# cup 占用百分比：us 用户使用；sy 系统使用；ni ；id 空闲/等待；wa 等待写入
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

# 内存占用：总内存；空闲内存；已经使用内存；缓存
KiB Mem : 16252328 total,   690012 free,  6821492 used,  8740824 buff/cache
KiB Swap:  8290300 total,  8290300 free,        0 used.  8294140 avail Mem 

# 进程详情 
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                          
22166 root      20   0  162784   3056   1588 R   1.0  0.0   0:00.70 top                                                                              
  733 polkitd   20   0 1697352 142076  21752 S   0.7  0.9 709:12.65 mongod   
  
# S 代表进程运行状态
# TIME+ 代表进程使用的总时间
```

更详细的参数介绍，这篇 [top 命令](https://www.cnblogs.com/peida/archive/2012/12/24/2831353.html 'Linux 命令详解') 写的非常详细，可以参考。

注意：僵尸进程会占用系统资源，影响正常程序运行，所以要及时处理掉僵尸进程。

```shell
# 使用 ps 过滤查找 stat 状态为 Z/z 的进程
$ ps -A -ostat,ppid,pid,cmd | grep -e '^[Zz]'

# 使用 kill 杀掉进程
$ kill -HUP 1234
```

##### 系统负载测试

新建两个服务器窗口

- 窗口 1：运行 top 命令，实时观察系统负载数据
- 窗口 2：运行测试命令

```shell
# yes 命令的作用是输出指定字符串，直到 yes 进程被杀死
# 运行 30 秒后获取到 yes 进程并杀死
$ { yes > /dev/null & } && sleep 30 && ps -ef | grep yes | awk '{print $2}' | xargs kill
```

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211214111307598.png)

切换到 top 命令窗口，发现 yes 命令启动的进程已经开始工作，用户态 CPU 使用率提升，系统负载增加，30 秒后 yes 进程被自动杀死，各项数据恢复。

再做一次更直观的测试，通过 for 循环启动与 CPU 核数相同的进程数运行 yes 命令。

```shell
$ for i in $(seq 0 $(($(cat /proc/cpuinfo | grep processor | wc -l) -l))); do taskset -c $i yes > /dev/null & done && sleep 30 && ps -ef | grep yes | awk '{print $2}' | xargs kill
```

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211214112032835.png)

这次发现 CPU 占用率已经接近百分百了，此时再访问该机器下的其他服务，响应就会相对变慢。说回工作，在测试过程中发现业务响应很慢，除了排查网络原因外，还可以登录服务器，通过 top 命令查看是否有其他进程导致 CPU 占用率过高。

### 内存

```shell
$ free -h
       total  used  free  shared  buff/cache  available
Mem:   15G    6.5G  674M  724M    8.3G        7.9G
Swap:  7.9G   0B    7.9G
```

- total：总物理内存
- used：已经使用的内存
- free：空闲可用内存
- shared：多进程共享内存
- buff/cache：读写缓存内存，当 free 不足时，内核会讲该部分内存释放
    - buffer：即将要被写入磁盘的
    - cache：即将从磁盘中读出来的
- available：还能被应用程序使用的物理内存


### IO

Input / Output

##### 硬盘 IO

通过 `iostat` 工具可以对系统的磁盘操作活动进行监视。

```shell
# 安装
$ yum install sysstat

# 使用
# 参数 1 代表每秒刷新一次
# 后面可以再跟一个数字，表示共刷新几次
$ iostat 1
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.00    0.00    0.00  100.00
           
# 机器硬盘信息
# tps：数据每秒传输次数；每秒从设备读写的数据量（读写速度）；读写的总数据量
Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.00         0.00         0.00          0          0
sdb               0.00         0.00         0.00          0          0
sdc               0.00         0.00         0.00          0          0
```

参数说明：

- %user：CPU 处在用户模式下的时间百分比
- %nice：CPU 处在带 NICE 值的用户模式下的时间百分比
- %system：CPU 处在系统模式下的时间百分比
- %iowait：CPU 等待输入输出完成时间的百分比，如果该值过高，表示硬盘存在 I/O 瓶颈
- %steal：管理程序维护另一个虚拟处理器时，虚拟 CPU 的无意识等待时间百分比
- %idle：CPU 空闲时间百分比，如果该值过高，表示 CPU 较空闲

**备注：**

如果 `%idle` 值高但系统响应慢时，有可能是 CPU 等待分配内存，此时应加大内存容量；`%idle` 值如果持续低于 10，那么系统的 CPU 处理能力较低，表明系统中最需要解决的资源是 CPU。



重新启动一个窗口，通过两条命令模拟 IO 操作：

```shell
# 写
$ dd if =/dev/zero bs=1024 count=4096000 of=test.iso

# 读
$ dd if=test.iso bs=64k | dd of=/dev/null
```

- bs：设置读/写缓冲区的字节数
- /dev/null：空设备，特殊的设备文件，丢弃一切写入其中的数据
- /dev/zero：特殊文件，当被读取时，提供无限的空字符



观察 `iostat` 数据情况，通过 `dd` 命令写入，由于是系统操作，所以用户态占用并不是很高，system 的 CPU 占用率升高。

![写操作](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211214125822107.png)



##### 网络 IO

使用 `iftop` 工具进行实时流量监控。

```shell
# 安装
$ yum install iftop

# 使用
$ iftop

# 测试向服务器发送大文件下载请求
```

![iftop 命令界面](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211214230008234.png)

- TX：发送流量
- RX：接收流量
- TOTAL：总流量
- cum：运行 iftop 到目前时间的总流量
- peak：流量峰值
- rates：分别表示过去 2s、10s、40s 的平均流量



### 总结

当请求服务器时无响应，首先检查本地网络、服务器网络是否正常，其次检查服务是否部署成功并正常运行。

当请求服务器返回响应过慢时，除了检查网络问题外，还需要关注一下服务器性能问题，通过性能统计命令来初步排查原因：

1、查看 CPU 占用率是否过高；

2、查看内存空间是否充足；

3、检查硬盘读写是否过慢过多；

4、检查网络流量传输，是否存在大流量堆积，导致超时响应请求。