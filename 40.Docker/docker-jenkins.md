> Docker 搭建 Jenkins 服务
> - 从零搭建服务
> - 历史服务迁移至 docker



### 一、从零搭建服务

##### 拉取镜像

```bash
# lts: Long Term Support
$ docker pull jenkins/jenkins:lts
```
注意注意注意！！！

默认镜像 `jenkins:latest` 版本已经废弃，拉取时手动指定 lts 长期维护版本。
![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220105105134363.png)

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/SkisPOBwErLmpH8I8hH9cgfwde07f5Gh.gif)


##### 启动容器 

需要挂载的内容太多，将容器启动命令封装为 shell 脚本，方便使用。

```bash
#!/bin/bash

name=jenkins

if [[ -n $(docker ps -q -f "name=^$name$") ]];
then
    docker rm -f $name;
fi

# 将宿主机 docker 挂载进来，否则 Jenkinsfile agent docker
# 报错：docker: not found
docker run -d --name $name \
    -p 8080:8080 -p 50000:50000 \
    --restart=always \
    -u root \
    --privileged=true \
    -v $(pwd)/volumes/jenkins_home:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock  \
    -v $(which docker):/bin/docker \
    -v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 \
    -v /var/lib/docker/tmp:/var/lib/docker/tmp \
    -e JAVA_OPTS=-Duser.timezone=Asia/Shanghai \
    jenkins/jenkins:lts
```

运行脚本

```bash
# 添加脚本可执行权限
$ chmod +x jenkins.sh
$ sh jenkins.sh
02e3239e020a98acd74af4a59b7ca53bb6e6e0fe6a135608056f7130edb5fe09
```

执行`docker ps`命令，发现容器并没有启动起来，查看容器运行日志。

```bash
$ docker logs -f jenkins
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```

原因是`jenkins`用户创建文件失败，没有写入权限，添加权限后重试。

```bash
# 添加挂载目录的可操作权限
# 方法一：添加 jenkins 用户的可读写权限，jenkins uid = 1000
$ chown -R 1000 ./volumes/jenkins_home
# 方法二：允许所有用户可读写挂载目录
$ chmod 777 ./volumes/jenkins_home

$ sh jenkins.sh
6b0c20ed3dd952dff85b4da2207267c597ce4851b13727059d6b807c70bcc340
```

浏览器输入服务器 IP:8080 即可访问服务，等待初始化完成，来到解锁服务页面，需要输入管理员密码。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220105101425024.png)



密码获取方法一：查看容器日志

```bash
$ docker logs -f jenkins
...
# 管理员密码
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

48fa9c4cd9f24745a87dc19a44df5796

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
```

密码获取方法二：查看初始化密码文件

因为 jenkins_home 被挂载到宿主机上，所以在容器内或宿主机挂载目录里都是可以查看密码文件的。

```bash
# 容器内查看默认密码
$ docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
48fa9c4cd9f24745a87dc19a44df5796

# 宿主机挂载目录查看默认密码
$ cat $(pwd)/volumes/jenkins_home/secrets/initialAdminPassword
48fa9c4cd9f24745a87dc19a44df5796
```

界面按照提示一步一步来，重设管理员密码，跳过插件安装，完成服务搭建。

![欢迎页](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211224102547974.png)



### 二、历史版本迁移

复制原服务下的 jenkins_home 目录，压缩后上传到新服务器，在新服务器上解压到自定义位置，修改容器启动脚本内的挂载目录，启动容器。

服务启动后出现报错：
```
java.nio.file.AccessDeniedException: /var/jenkins_home/secret.key
...
Failed to fully read /var/jenkins_home/secret.key
```
权限问题，解决办法：
```bash
$ chown -R 1000 jenkins_home
```
重启容器，正常访问，原历史数据全部恢复。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/qkJnJgz7G9LtM5Nx9L09ot9eTdPUDPNS.gif)

