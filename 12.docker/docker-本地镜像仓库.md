## Docker registry 

### 简介

Docker Registry 是存储 Dcoker Image 的仓库，运行 push、pull、search 时，是通过 Docker daemon 与 docker registry 通信。有时候使用 Docker Hub 这样的公共仓库可能不方便，我们可以通过 registry 创建一个本地仓库。



### 运行

```shell
# 搭建仓库的主机运行
$ docker run -d -p 5000:5000 -v ${PWD}/registry:/var/lib/registry --restart always --name registry registry:2.7.1
```

```shell
# 其他机配置下镜像仓库
$ vim /etc/docker/daemon.json
{
	"insecure-registries":["{仓库主机的IP地址和端口号}"]
}
$ systemctl restart docker  # 重启下docker
```



### 演示

```shell
# 搭建仓库的主机：先从 docker hub 上随便下载一个镜像
$ docker pull nginx:1.18.0
# 打上带主机 IP 地址的标签
$ docker tag nginx:1.18.0 {仓库地址}/nginx:1.18.0
# 推送到内网仓库下
$ docker push {仓库IP:port}/nginx:1.18.0

# 其他机器
$ docker pull {仓库IP:port}/nginx:1.18.0
# 查看下载的镜像，就是本地仓库的镜像
$ docker image
```

```shell
# 构建镜像并推送到本地仓库上
$ docker build -t {仓库IP:port}/flask-web:1.0 .
```

通过搭建本地仓库，下载镜像后 push 到本地仓库，其他内网机器 pull 的话速度会有所提升。

一般公司业务不方便放到 docker hub 上，可以通过内网仓库提高隐私性。

