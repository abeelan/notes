> 汇总工作中用到的 Linux 基础命令

### SSH

```shell
# 登录服务器，输入密码，密码不可见
# 默认端口号为 22，可不写
$ ssh -p 22 username@host
```
登录成功后，前缀：

- `$` 代表普通用户 
- `#` 代表管理员用户

其他命令：

- `sudo`：以预设身份执行命令，默认为 root 用户
- `exit`：退出系统

### 查看帮助信息

这两个命令用到的频率比较高，毕竟那么多的参数功能是记不住的。

- --help

```shell
$ ls --help
```

- man

```shell
$ man ls
# 空格翻下一页，d 返回上一页
# 回车翻行
# q 退出 
```

英文是硬伤的同学可以安装汉化版文档

[manpages-zh](https://github.com/man-pages-zh/manpages-zh "manpages-zh")


### 磁盘管理

- 查看内存使用情况：`$ free`
- 查看外存使用情况：`$ df -h`



### 文件管理

- 查看文件信息：`$ ls -al`(list)
- 切换工作目录：`$ cd`(chage directory)
- 显示当前路径：`$ pwd`
- 创建新目录：`$ mkdir`
- 创建新文件：`$ touch`
- 删除文件或目录：`$ rm -rf`
- 拷贝文件：`$ cp`
- 剪切文件：`$ mv`
- 建立链接文件：`$ ln`
    - 软链接：添加 `-s` 参数，不占用磁盘空间，原文件删除后，链接失效
    - 硬链接：占用磁盘空间，原文件删除后链接文件依然存在，不影响使用；只能链接普通文件，不能链接目录
- 查找文件：`$ find`
- 查看文件内容
    - `cat`：查看整个文件
    - `less`：按屏展示
      - 空格下一页
      - d 上一页
      - q 退出
    - `more`：同 less，增加百分比展示
    - `head`：从文件头部开始，默认展示前 10 行，通过 `-n` 参数指定展示行数
    - `tail`：同 head，从文件尾部开始
    - `diff`：比较两个文件的差异
- 打包压缩：`tar`
    - `-zcvf`：压缩
    - `-zxvf`：解压缩
    - `-C`： 指定路径
- 修改文件权限：`chmod 777 filename`
    - r：read，读权限，4
    - w：write，写权限，2
    - x：execute，执行权限，1
    - -：无权限，0
- 文件属性，如图：

![图片来自测试人论坛](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211213172505682.png)

### 文本处理

- 文本编辑：`vi/vim`（有编程能力）
    - 跳到开始行：gg
    - 跳到末尾行：G（shift + g）
    - 行首跳行尾：$
    - 行尾跳行首：<
- 屏幕输出：`echo`
- 输出重定向：`>`

### 进程管理
- `ps`：进程列表快照
- `top`：交互式进程观测
- `kill`：结束进程
  - 强制杀死进程 `$ kill -9`
- `fg`： 进程切换到前台（可交互）
- `bg`： 进程切换到后台
- `ctrl + z` 快捷键，挂起进程

##### PS 命令
```shell
# 获得当前所有进程列表

$ ps -ef  # UNIX 风格参数，单减号
# UID：启动这些进程的用户
# PID：进程 ID
# PPID：进程ID
# STIME：进程启动时的系统时间
# TTY：进程启动时的终端设备
# TIME：运行进程累计时间 
# CMD：启动进程时的名称

$ ps aux  # BSD 风格，无符号
# USER PID 
# %CPU 
# %MEM    
# VSZ：进程在内存中的大小,以千字节(KB)为单位，虚拟内存
# RSS：进程在未换出时占用的物理内存，保留内存
# STAT：代表当前进程状态的双字符状态码
# TIME：进程运行时间
# COMMAND：启动进程时的命令

$ ps --pid pidlist # GUN 风格，双减号

# 自定义输出指标
$ ps -o pid,ppid,psr,thcount,tid,cmd -M
# psr：运行在哪个 cpu 上
# thcount：线程数
# tid：线程号
# cmd：对应的执行命令
```

### 网络

- `ifconfig`：查看各个网卡信息
- `ping`：测试远程主机联通性
    - `-c`：设置次数
    - `-i`：设置时间间隔
- `netstat`：查看网络信息
    - `-at`：列出所有 tcp 相关选项
    - `-au`：列出所有 udp 相关选项
    - `-n`：以数字形式显示地址和端口号
    - `-p`：显示进程的 pid 和名字



### 定时任务

```shell
# 启动服务
$ service crond start 
# 停止服务
$ service crond stop
# 重启服务
$ service crond restart 
# 查看服务状态
$ service crond status
# 设置定时任务
$ crontab -e
```