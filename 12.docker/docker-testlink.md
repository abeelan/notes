## 搭建测试用例管理平台 Testlink

### Testlink 简介

Testlink 基于 WEB 的测试用例管理系统，主要功能是测试项目管理、产品需求管理、测试用例管理、测试计划管理、测试用例的创建、管理和执行，并且还提供了统计功能。

### 部署数据库

```shell
# 创建一个名为 testlink 的容器网络
$ docker network create testlink
# 查看当前存在的容器网络
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE      备注（自己加的）
4deb334c55f6        bridge              bridge              local      桥接网络
cf4fbaac0160        host                host                local      主机网络
50027393c74d        none                null                local
6ac9853f1eab        testlink            bridge              local      刚才新建的网络
# 运行数据库，如果本地没有该镜像的话会自动去下载
$ docker run -d --name mariadb -e MARIADB_ROOT_PASSWORD=mariadb -e MARIADB_USER=bn_testlink -e MARIADB_PASSWORD=bn_testlink -e MARIADB_DATABASE=bitnami_testlink --net testlink -v ${PWD}/mariadb:/bitnami bitnami/mariadb:10.3.22

Unable to find image 'bitnami/mariadb:10.3.22' locally
10.3.22: Pulling from bitnami/mariadb
...
74dbf92d3ddca4d241f2b9022890445820f7e26e5324c352c90983515379bd65
```

### 部署 Testlink

```shell
# 运行 testlink
$ docker run -d -p 80:8080 -p 443:8443 --name testlink -e TESTLINK_DATABASE_USER=bn_testlink -e TESTLINK_DATABASE_PASSWORD=bn_testlink -e TESTLINK_DATABASE_NAME=bitnami_testlink --net testlink -v ${PWD}/testlink:/bitnami bitnami/testlink:1.9.20

Unable to find image 'bitnami/testlink:1.9.20' locally
1.9.20: Pulling from bitnami/testlink
...
5e766b7f386db5c0bdb3adcb14c1daea6f5f9517f56dfc9af556fbcb4c6e1bc9
```

在浏览器访问本地 IP 地址，即可正常浏览网页。

默认用户名：user；默认密码：bitnami。

