> 关于 docker 的一些问题。



### 问题 1：在 selenium grid 中，多个 node 是在同一机器上，依然是在单机上执行自动化测试，如何将这些 node 分布到不同的机器上？

```shell
$ docker exec -it chrome bash
$ env | grep HUB
HUB_PORT_4444_TCP_ADDR=172.17.0.3
HUB_PORT_4444_TCP_PORT=4444
```

`--link hub` 的原理是将目标容器的 HUB 信息以环境变量的形式注入到容器内，如果两个容器不在同一机器上无法使用 --link 参数。

那么，在其他机器上启动容器的时候，可以通过 -e 参数，填写上面获取的 IP 地址和端口，以设置环境变量的方式传递给容器信息，即可实现多机器运行 node 节点。



### 问题 2：在 docker selenium 的开源镜像中，为什么不支持 IE 浏览器的镜像？

因为容器并不虚拟化自己的内核，一台机器上所有的容器都使用宿主机的内核。

而 docker 只能安装在 linux 系统上，因为容器的底层逻辑需要 linux 内核的能力，而 IE 浏览器是使用 windows 内核驱动的，所以无法制作 IE 浏览器的镜像。

容器都是使用宿主机的内核，如果运行的软件挑内核或者内核版本，那么尽量不要用容器来运行。

windows 上之所以能运行 docker，是因为先安装了 linux 系统，然后再安装的 docker，微软已经在开发自己的 docker，但是目前还没有太多人使用。



###  问题 3：需要测试 toB 产品兼容不同的操作系统，比如不同版本的 centos、redhat、ubantu 等等，通过在宿主机下载这些镜像去安装软件，软件都安装成功，能不能认为测试通过？

不能；因为容器使用的内核是宿主机的内核，比如在 ubantu 上安装 centos 镜像，运行内核依然是 ubantu，centos 镜像只是安装了一些 centos 的软件供用户使用而已，所以在容器安装成功，不能真的代表在 centos 操作系统上运行成功。

所以总结下来任何对内核有要求的场景尽量不要使用容器测试。



### 问题 4：现在需要排查一个容器中的网络问题，但是容器没有安装任何的网络排查工具，也无法通过网络下载工具，这个时候要怎么做？

可以启动一个带有网络排查工具的容器，然后通过 container 网络模式将新创建的容器挂载到要排查的容器上。这样两个容器的网络就是互通的，就可以抓取到目标容器的网络流量。

还有一种方法就是不用启动新的容器，而是直接让宿主机切换到容器网络的名称空间

```shell
$ docker inspect jenkins
# 查找容器 PID -> 24171

# 进入该 pid 对应的地址
$ cd /proc/24171

# ns = namespace 名称空间
$ cd ns

# 在宿主机上切换到容器的名称空间
# -n 代表切换 network namespace
$ nsenter --help
$ nsenter -t 24171 -n

# 查看只有两个网络设备，代表成功切换到容器网络
$ ifconfig
# 退出网络名称空间，可以看到宿主机是有多个网络设备的
$ exit
```



### 问题 5: 解释一下容器网络的运行原理（三种连接方式）

##### 桥接模式（创建虚拟网卡）

![image-20210826234228534](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210826234228534.png)

![image-20210827003822267](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210827003822267.png)

![image-20210826235559874](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210826235559874.png)



##### host 模式（不创建虚拟网卡，直接使用宿主机网络）

![image-20210826235611583](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210826235611583.png)



##### 容器网络模式（打破容器间的界限，应用场景较多，很关键。）

![image-20210826235656404](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210826235656404.png)

通过 container 模式进行 mock server 原理图

![image-20210827000417357](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210827000417357.png)

通过 container 模式进行多服务日志收集上传原理图

![image-20210827004238174](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210827004238174.png)

通过 container 模式，进程共享，可以进行故障注入

sandbox 连接到容器 A 的进程内，A 进程运行的同时，sandbox 进行一些故障注入。

![image-20210827004339146](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210827004339146.png)



### 问题 6：dockerfile RUN 指令，执行多个命令时，下面那种方式更好一些？

```docker file
### 方式一 ###
FROM centos:7
RUN yum install -y wegt vim git
RUN yum install -y wget
RUN yum install -y git

### 方式二 ###
FROM centos:7
RUN yum install -y wegt vim git \
 && yum install -y wget \
 && yum install -y git
```

用两种方式分别构建镜像，查看 dockerfile 的执行过程

##### 使用方式一构建镜像

```shell
$ docker build -t tmp .
# . 代表存放 dockerfile 的目录，开始构建时，先将当前目录下的所有文件打包发送给 docker 进程
# 所以 dockerfile 独立保存在一个目录下而且该目录下不要放无关的文件
Sending build context to Docker daemon 4.608kB

# 每执行安装一个命令都会重新新建一个临时容器，安装完成后会移除该中间容器
Step 3/3: yum install -y git
---> Running in xxxxxx
Removing intermediate container xxxxxx

# 再次进行构建，会自动使用缓存，如果修改 dockerfile，哪行被修改重新执行哪行，其他行依然会使用缓存。

# 优点：
# 拆分步骤后，可以复用缓存，提高效率

# 缺点：
# 镜像是分层的，一个 RUN 指令就是一层，每多一层就会有一层的io性能开销，如果多个 RUN 指令会影响性能。
```

##### 为什么会提升性能开销？

![image-20210827103626572](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210827103626572.png)

解释：当前 X 目录下有 A 文件，Y 目录下有 B 文件，如果管理员需要分别对 AB 两个文件操作的话，需要切换目录，但是如果文件多的话，切换目录是非常麻烦的，所以，docker 底层是文件共享系统 overlay2，通过 `docker info` 可以查看；

共享文件系统 Z 里面会有 AB 两个文件，对  A 文件添加字符串 hello 后，X 目录下的 A 文件也会同步修改，就不用切换目录；

对 Z 目录的 B 文件添加字符串 world 后，Y 目录下的 B 文件不会同步修改，而是在 A 目录下面新建一个文件，记录 B 文件的修改；

这是因为 docker 文件系统只会对第一个文件开发读写权限，其他文件全部仅可读，视图文件层 Z 目录的 B 文件，其实是通过 Y 目录的 B 文件读内容 merge X 目录 B 文件的更改，展示到 Z 目录下。

那么，文件过多的话，merge 操作会增加，所以会增加系统读写开销，拖慢性能。



##### 使用方式二再次构建镜像

```dockerfile
# 缺点：
# 使用方式二，修改 RUN 指令内的任意一个东西，都会整条命令全部重新执行，没有使用缓存。

# 优点：
# 节省性能开销
```

##### 总结：

开发 dockerfile 过程中，可以使用多个 RUN 指令的方式，这样便于调试，提高效率；

如果 dockerfile 已经开发完成，需要将 RUN 指令合并，减少层数，减少 IO 性能开销。



### 问题 7: 自动化测试环境搭建及使用，一种简单的模式

在 dockerfile 内定义安装 python 环境，设置启动脚本，即容器启动就会运行 entrypoint.sh 脚本

![image-20210827120136493](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210827120136493.png)

容器启动即去拉取最新代码，然后运行测试程序。

![image-20210827120238266](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210827120238266.png)

