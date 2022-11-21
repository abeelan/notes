## 搭建镜像仓库

```shell
$ docker pull registry:2
$ docker run -d -p 5000:5000 -v /usr/local/registry:/var/lib/registry --restart=always --name registry registry:2

# --restart=always 代表容器万一挂掉，docker 会永远重启容器
```

```shell
# DEMO
# 先下载一个小镜像
$ docker pull busybox
# 上传镜像前需要更新名字，命名规范 ==> IP地址:端口号/名称:版本号；
# 上面 pull busybox，其实应该是 library/busybox:latest
# library 是docker默认仓库地址，版本号不写默认是latest，代表最新版本
$ docker tag busybox localhost:5000/busybox:v1.0
$ docker push localhost:5000/busybox:v1.0
# 通过命令查看镜像仓库
$ curl http://localhost:5000/v2/_catalog
```

```shell
# 从一个运行的容器制作镜像
$ docker commit registry localhost:5000/myimage:v1.0
$ docker images | grep myimage
$ docker push localhost:5000/myimage:v1.0
$ curl http://localhost:5000/v2/_catalog
# 这种方法不推荐，别人拿到这个镜像，如果要做一些扩展，不知道安装了什么软件、设置权限、配置等等
```

```shell
# 为了解决上面的问题，推荐使用 dockerfile
# 将制作镜像的过程脚本化
```

