> 接口功能自动化测试项目集成到 jenkins 实战



### 安装 python

在 jenkins 容器内安装 python 执行环境

```shell
$ docker exec -it -u root jenkins /bin/bash
$ cat /etc/issue  
Debian GNU/Linux 1
# 根据 debian 版本更新下载源
# https://blog.csdn.net/lan_yangbi/article/details/86720257
$ apt-get update

# 下载安装包
$ apt-get install wget
$ wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
$ tar -zxvf Python-3.6.8.tgz

# 编译安装
$ cd Python-3.6.8
$ ./configure --prefix=/opt/python3 --with-ssl
$ apt-get -y install gcc automake autoconf libtool make
$ apt-get -y install make*
$ apt-get -y install zlib*
$ apt-get -y install openssl libssl-dev
$ apt-get install sudo
$ make && make install 

# 软链
$ ln -s /opt/python3/bin/python3.6 /usr/bin/python
$ ln -s /opt/python3/bin/pip3 /usr/bin/pip
```



### 配置从 git 拉取项目

在 Source Code Management 内选择 Git，添加仓库地址；

```shell
# 仓库地址
$ https://github.com/princeqjzh/iInterface_python.git
```

在 Additional Behaviours 内增加 check out to a sub-directory，目的是在工作目录创建一个项目地址



### 构建命令

在 Bulid 内添加 Execute shell，写入构建要执行的命令

```shell
$ cd iInterface_python
$ sudo pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
$ python -m pytest -sv test/weather_test.py
```

 由于执行命令的是 jenkins 用户，无法执行 sudo 命令，这里还需要设置一下 jenkins 用户免密使用 sudo。

```shell
# root 用户
$ sudo vi /etc/sudoers
$ echo 'jenkins ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

直接运行 pytest 会提示：bash: pytest: command not found

因为系统存在默认的 python2 和 pip2 版本，所以

### 配置 allure 报告

##### 容器环境内安装 allure commandline

```shell
$ cd /opt/python3
$ wget https://registry.npmjs.org/allure-commandline/-/allure-commandline-2.13.0.tgz
$ mkdir allure
$ tar -zxvf allure-commandline-2.13.0.tgz -C ./allure/

# 软链
$ find / -name allure
/opt/python3/allure  # 自己创建的目录
/opt/python3/allure/package/bin/allure
/opt/python3/allure/package/dist/bin/allure  # 软链这个
$ ln -s /opt/python3/allure/package/dist/bin/allure /usr/bin/allure
$ allure --version
2.13.0
```

##### jenkins 安装 allure 插件

在 插件市场进行下载安装：[Allure Jenkins Plugin](https://plugins.jenkins.io/allure-jenkins-plugin)

step 1：在项目构建后添加步骤 Post-build Actions - Allure Report

![image-20210829100231037](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210829100231037.png)

step 2：添加 allure 到 jenkins 的全局环境变量内，这里直接添加 allure/package/bin目录下的allure 会报找不到node 的错误，换成dist/bin 目录下的就可以成功运行，所以在环境变量这里填写 dist 目录。

上面软链成 dist 目录也是因为这个原因。

![image-20210829103813732](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210829103813732.png)

step 3：还需要修改一下 execute shell 内的 pytest 命令

```shell
# 添加 allure 报告地址，必须与上面配置的 path 一致
$ python -m pytest -sv test/weather_test.py --alluredir ./allure-results
```



### 构建项目

进行构建项目，即可看到 allure 报告展示。

