`YApi` 是比较好用的接口管理平台，通过 docker-compose 搭建记录。



### 准备工作

> CentOS Linux release 7.9.2009 (Core)
>
> Docker 20.10.8



### 搭建

##### 1. 创建容器网络

```shell
$ docker network create --driver bridge --subnet=10.10.0.0/16 --gateway=10.10.0.1 yapi
```

目的是保证 mangodb 和 yapi 两个容器间可以互相通信。

```shell
# 创建工作目录并进入
$ mkdir yapi-compose && cd yapi-compose
```



##### 2. 创建 mongodb-compose

```shell
$ vim mongodb-compose.yml
```
```dockerfile
version: '3'
services:
  mongodb:
    image: mongo:4.4.4       # 镜像名
    container_name: mongodb  # 容器名
    volumes: # 数据挂载目录；本地目录:容器目录        
      - /data/docker/volumes/yapi-mongodb:/data/db  
    ports:
      - 27017:27017  # 端口，宿主机:容器
    # environment:  
      # - COMPOSE_PROJECT_NAME=yapi
    restart: always

# 设置默认网络
networks:
  default:
    external:
      # 创建的容器网络名
      name: yapi  

# ESC => :wq ==> Enter
```



### 3. 创建 yapi-compose

```shell
$ vim yapi-compose.yml
```
```dockerfile
version: '3'
services:
  yapi:
    image: jayfong/yapi:latest                  # 镜像名
    container_name: yapi                        # 容器名
    privileged: true                            # 赋予root权限
    ports:
      - 3000:3000                               # 端口，宿主机:容器
    environment:
      - YAPI_ADMIN_ACCOUNT=admin@easou.com      # 登入账号
      - YAPI_ADMIN_PASSWORD=admin               # 登入密码
      - YAPI_CLOSE_REGISTER=true                # 关闭注册功能
      - YAPI_DB_SERVERNAME=mongodb              # mongodb 数据库地址
      - YAPI_DB_PORT=27017                      # mongodb 端口
      - YAPI_DB_DATABASE=yapi                   # mongodb 数据库名
      - YAPI_MAIL_ENABLE=false                  # 不启用邮箱
      - YAPI_LDAP_LOGIN_ENABLE=false            # 不启用 loap 登入
      - YAPI_PLUGINS=[]                         # 插件
      # - COMPOSE_PROJECT_NAME=yapi
    restart: always

# 设置默认网络
networks:
  default:
    external:
      name: yapi
```



### 4. 构建容器

```shell
$ docker-compose -p mongodb -f mongodb-compose.yml up -d
$ docker-compose -p yapi -f yapi-compose.yml up -d
```



### 5. 访问

浏览器输入：http://{ip}:3000



### 最后

如果遇到问题，可以通过查看日志的方式定位到问题。

```shell
$ docker logs -f {container_id}
```
<br>

---

#### 安装时遇到的错误记录

##### 错误 1: 

```text
MongoDB 5.0+ requires a CPU with AVX support
```

问题：mongo 镜像如果不指定版本，会自动下载 latest 版本（5.0+）会出现上面的报错，且容器一直是 `restarting` 的状态。

解决：理论安装上 5.0 以下的版本都可以，我这里使用 `4.4.4` 版本安装成功。



##### 错误 2：

```text
Error [ValidationError]: user validation failed: username: Path `username` is required.
```

yapi-compose.yml 文件内，账号必须为邮箱格式，我刚开始图方便设置账号为 admin，一直无法登录，查看日志后发现用户名写入 db 失败，由于没有指定 emailAddress，默认使用用户名代替，报校验失败的错误。

如果成功的话，构建后，日志内会出现如下提示：

```
始化管理员账号成功,账号名："admin@easou.cn"，密码："admin"
```



##### 错误 3:

```shell
$ mv mongodb-compose.yml docker-compose.yml
$ docker-compose -d up
$ mv docker-compose.yml mongodb-compose.yml

$ mv yapi-compose.yml docker-compose.yml
$ docker-compose -d up
$ mv docker-compose.yml yapi-compose.yml
WARNING: Found orphan containers (name1, name2) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
```

注意：刚开始我使用这种方式构建，报错后网上搜索了下，通过 -p 指定项目名称的方式解决。

后来查文档，发现 -p 通过环境变量来指定也行，理论上将 `COMPOSE_PROJECT_NAME=xx` 放在 conpose 文件内，应该也可以，由于服务已经部署成功，没有再测试，参数我补充在上面的 compose 文件中了，已注释。



##### 错误 4:

```
Error: EACCES: permission denied, mkdir '/sys/fs/cgroup/memory/safeify'
```

无创建文件权限，需要以 root 用户启动容器，使用该参数，container内的root拥有真正的root权限，否则， container 内的 root 只是外部的一个普通用户权限。

```dockerfile
# yapi 的 compose 内添加如下设置
privileged: true

# 或者启动容器时加上这个参数
--privileged=true
```



### 最新

通过一个 compose 就起来了。

```yaml
# compose.yml
version: '3'

services:
  yapi-web:
    image: jayfong/yapi:latest                  # 镜像名
    container_name: yapi-web                    # 容器名
    ports:
      - 3000:3000                               # 端口，宿主机:容器
    volumes:
      - /data1/docker/yapi/volumes/sandbox.js:/yapi/vendors/server/utils/sandbox.js
    environment:
      - YAPI_ADMIN_ACCOUNT=admin@easou.com      # 管理员账号
      - YAPI_ADMIN_PASSWORD=admin               # 管理员密码
      - YAPI_CLOSE_REGISTER=true                # 关闭注册功能
      - YAPI_DB_SERVERNAME=172.20.0.2
      - YAPI_DB_PORT=27017                      # MongoDB 服务端口
      - YAPI_DB_DATABASE=yapi                   # MongoDB 数据库名
      # - YAPI_DB_USER=root                     # 登录 MongoDB 服务的用户名
      # - YAPI_DB_PASS=root                     # 登录 MongoDB 服务的用户密码
      - YAPI_MAIL_ENABLE=false                  # 不启用邮箱
      - YAPI_LDAP_LOGIN_ENABLE=false            # 不启用 loap 登入
      - YAPI_PLUGINS=[]                         # 插件
    depends_on:
      - yapi-mongo
    links:
      - yapi-mongo
    restart: unless-stopped

  yapi-mongo:
    image: mongo:latest
    container_name: yapi-mongo
    volumes:
      - /data1/docker/yapi/volumes/mongodb:/data/db
    expose:
      - 27017
    restart: unless-stopped
```

```bash
$ docker-compose up -d
```

