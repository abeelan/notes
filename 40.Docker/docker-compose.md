## Dcoekr-compose

### 简介

Docker-compose 是用于定义和运行多容器的 Docker 应用程序的工具。通过 compose，可以使用 yaml 文件来配置应用程序的服务。compose 的使用一般分为三步：

1. 使用 Dockerfile 定义应用程序的环境，以便可以在任何地方复制它；
2. 在 docker-compose.yml 中定义组成应用程序的服务，以便他们可以在隔离的环境中一起运行；
3. 运行 docker-compose up，然后 compose 启动并运行您的整个应用程序。



### 安装

macOS、windwos 上的 Desktop 版本，默认已经安装

```shell
# 验证
$ docker-compose version
docker-compose version 1.23.2, build 1110ad01
docker-py version: 3.6.0
CPython version: 3.6.6
OpenSSL version: OpenSSL 1.1.0h  27 Mar 2018
```

CentOS 安装

```shell
# 根据服务器型号安装稳定版本
# -L 跟随链接重定向
$ curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 更改权限
$ chmod +x /usr/local/bin/docker-compose

# 查看版本
$ docker-compose version
Docker Compose version v2.2.2
```



### 常用命令

```shell
# 查看配置
$ docker-compose config

# 后台启动
$ docker-compose up -d
# 指定名称
$ docker-compose -p "selenium-grid" up -d
# 指定配置文件启动
$ docker-compose -f docker-compose.yml up -d

# 构建镜像；仅构建，不启动容器
$ docker-compose build

# 下载镜像，仅拉取镜像
$ docker-compose pull

# 查看运行状态
$ docker-compose ps 

# 查看容器内运行的进程
$ docker-compose top

# 启动/停止/重启/暂停/恢复
$ docker-compose start
$ docker-compose stop
$ docker-compose restart
$ docker-compose pause
$ docker-compose unpause

# 停止并删除已创建的容器网络及容器
$ docker-compose down
```



### 演示

创建一个 web 服务，统计当前服务被访问的次数。项目文件目录如下：

- docker-compose.yml
- .env
- dockerfile
- app.py



1、创建 docker-compose 配置文件

```yml
# 指定 docker-compose API 版本
version: '3'
services:
  web:
    build:
	  context: .
	depends_on:
	  - redis
	ports:
	  - "5000:5000"
	
  redis:
    image: ${REDIS_VERSION}
	restart: always
```



2、创建环境配置文件

```ini
# .env
REDIS_VERSION=redis:alpine
```

验证配置文件

```shell
$ docker-compose config
# context：. 会变成实际工作目录
# 镜像变量也会自动替换为 env 文件内配置的值
```



3、创建 dockerfile

```shell
# Dockerfile
FROM python:3.7-alpine
LABEL maintainer="test"

# 工作路径设定为 /code
WORKDIR /code

# 创建环境变量给 Flask 使用
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0

# 复制 app.py 到容器内的 /code 目录
COPY app.py /code

# 安装 python 库
RUN pip install flask && pip install redis

# 映射端口
EXPOSE 5000

STOPSIGNAL SIGTERM

# 为容器设置默认启动命令
CMD ["flask", "run"]
```



4、创建业务代码文件

```python
# app.py
# 实现统计访问次数
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
	retries = 5
	while True:
		try:
			return cache.incr("hits")
		except redis.exceptions.ConnectionError as e:
			if retries == 0:
				raise e
			retries -= 1
			time.sleep(0.5)

@app.route('/')
def hello():
	count = get_hit_count()
	return f"Hello World! I have been seen {count} times.\n"
```



##### 启动

```shell
$ docker-compose up -d
$ docker-compose logs -f
```



