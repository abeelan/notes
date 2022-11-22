>  docker 部署 Gitlab



##### 拉取镜像

```shell
$ docker pull gitlab/gitlab-ce:latest
```



##### 启动容器

```bash 
#!/bin/bash

NAME=gitlab
VOLUMES=/data1/docker/gitlab/volumes
HOST=`ifconfig eth1 | grep 'inet' | awk '{print $2}'`

if [[ -n $(docker ps -a -q -f "name=^$NAME$") ]];
then
    docker rm -f $NAME;
fi

docker run -d --name $NAME \
    --hostname $HOST \
    --publish 4433:443 \
    --publish 8880:80 \
    --publish 222:22 \
    --restart always \
    -v $VOLUMES/config:/etc/gitlab \
    -v $VOLUMES/logs:/var/log/gitlab \
    -v $VOLUMES/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest  
```

启动后，初始化非常慢，耐心等待

```shell
$ docker logs -f gitlab
...
```

查看初始化登录密码

```shell
$ docker exec -it gitlab bash
$ cat /etc/gitlab/initial_root_password
Password: GVmOPy613BipQ38T5xdEpjS/Q/yWCgnHq7KQJJyhHpI=

# 重置初始化密码
$ gitlab-rails console
Loading production environment (Rails 6.1.4.1)
irb(main):001:0> user=User.where(id:1).first
=> #<User id:1 @root>
irb(main):002:0> user.password='12345678'
=> "12345678"
irb(main):003:0> user.password_confirmation='12345678'
=> "12345678"
irb(main):007:0> user.save
=> true
```



### gitlab

##### 创建新项目

 New project -> Create blank project    

- 设置项目名称
- 公开访问

创建成功后，根据提示初始化代码目录

```shell
$ git init
$ git clone http://10.26.20.117:8880/root/tmp.git
Cloning into 'tmp'...
warning: You appear to have cloned an empty repository.
```



##### 删除项目

仓库 -> Settings -> General -> Advanced -> Expand -> Delete project



##### 与 Jenkins 打通

Jenkins 需要下载 gitlab 插件；

Dashboard -> Manage jenkins -> Configure System -> Gitlab，进行配置：

1、配置名称：自定义

2、Gitlab host URL：gitlab 的访问地址

3、credentials：全局凭证

- 在 gitlab 生成 token 填入，gitlab token 生成：gitlab -> 用户头像 -> edit profile -> Access Token -> 填写 token 名称、日期、权限 -> 创建

- 添加认证 -> kind（GitLab API token）-> 填入 API token

4、保存配置

5、项目配置内，构建触发器选择 gitlab webhooks

![image-20211231150621265](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211231150621265.png)

6、gitlab 进入项目内，Settings -> Webhooks -> 填写上面 Jenkins 构建触发器上面显示的地址

通过 **Push events**可以精确的控制触发分支名称，支持正则。



# Jenkins 和 gitlab 一直链接不上！

- 通过指定容器网络，网络可以互相 ping 通

    



![image-20210909143019105](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210909143019105.png)



mysql 启动脚本

![image-20210910141105573](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210910141105573.png)

