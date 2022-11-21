> Dockerfile 是由一系列指令和参数构成的脚本，一个 Dockerfile 里面包含了构建整个镜像的完整命令。



工作中，由于业务水平与技术选型不同，在使用 Docker 起服务时，需要定制项目镜像。通过两种方式：

- Docker commit
- Dockerfile



##### [Docker commit](https://docs.docker.com/engine/reference/commandline/commit "Docker commit doc")

基于一个运行状态的容器，在容器内安装项目运行所需要的环境依赖，再创建出一个新的镜像。

```bash
$ docker commit \
	-a "author" \
	-m "commit info" \
	container_id \
	image_name:tag
```

使用这种方式制作的镜像，对外不可解释，不知道镜像内的环境组成，不方便排查问题，可维护性差。



##### [Dockerfile](https://docs.docker.com/engine/reference/builder "Dockerfile doc")

Dockerfile 是一个文本文档，通过 **docker build** 可以执行 Dockerfile 中的一系列指令自动构建镜像。Dockerfile 的目录最好不要放其他文件，因为构建时会将该目录下的文件内容全部传输到 docker daemon，影响构建性能。

```shell
# -t 指定镜像名称和标签
# -f 指定 Dockerfile 文件路径
# . 代表当前路径，千万别忘记写
$ docker build -t pytest:v1 -f Dockerfile .
```

详细的说明请参考官方文档，推荐直接去看官方文档。下面记录 Dockerfile 常用的指令及其用法。



### 常用指令

指令不区分大小写，为方便区分指令和参数，约定指令统一大写。

```dockerfile
FROM    # 指定基础镜像，Dockerfile 必须以 FROM 指令开始（除 ARG）
LABEL   # 标签信息，作者、版本等；<key>=<value> <key>=<value> ...
ENV     # 容器启动的环境变量；<key>=<value> ...
ARG     # 仅作用于 Dockerfile 的环境变量，可以出现在 FROM 指令之前
USER    # 指定运行容器时的用户名或 UID，默认管理员用户，后续 RUN 也会使用指定用户
WORKDIR # 设置工作目录，作用于 RUN、CMD、ENTRYPOINT、ADD 指令
ADD     # 添加压缩文件到镜像中，自动解压缩
COPY    # 复制文件到镜像中
VOLUME  # 挂载目录
RUN     # 运行命令，每条 RUN 指令在当前基础镜像上执行，并且会提交一个新的镜像层
EXPOSE  # 对外暴漏端口号，可以被容器运行时的 -p 参数覆盖掉
CMD     # 容器启动时执行的默认命令
ENTRYPOINT   # 容器启动时指定参数
HEALTHCHECK  # 容器健康状态检查
```

编写 Dockerfile 的最佳实践移步官方文档，写的非常详细了。

[Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices "Best practices for writing Dockerfiles")

列出几点：

- 多行参数使用换行符

- 不要安装非必要的包
- 每个容器只干一件事，解耦应用程序
- 尽量减少镜像层数
- 充分利用构建缓存

镜像制作过程中，每个指令代表一层，镜像是由多层镜像累加起来的。指令会产生缓存，构建过程中会优先使用缓存，所以当开发镜像时，可以使用多个 RUN 命令，充分利用缓存，提高调试的效率。通过 `--no-cache` 可以人为避免使用缓存。

Dockerfile 开发完成后，用 && 符号把 RUN 指令合并，作为同一层镜像进行构建。因为镜像的分层原理，每多一层就会多一次 IO 开销，当指令过多，可能会影响系统性能。 



### 演示

使用 Dockerfile 构建 nginx 容器，添加容器健康检查。

```bash
# index.html
<h1>Hello Nginx!</h1>
```

```dockerfile
# Dockerfile
# 导入基础镜像
FROM nginx:latest

# 指定信息
LABEL Description="Nginx Testing services."

# 设置环境变量
ENV WELCOME "hello, nginx!"

# ARG 设置的环境变量只在 dockerfile 生效
ARG dir=/usr/share/nginx

# 设置工作目录
WORKDIR $dir/html

# 切换用户
USER root

# 执行安装命令
RUN apt-get -y update && apt-get install -y curl

# 复制 index.html 文件到 workdir 目录下
COPY index.html .

# 映射 80 端口
EXPOSE 80

# 此处 CMD 作为 ENTRYPOINT 的参数
CMD ["nginx", "-g", "daemon off;"]

# 检查容器健康，通过访问 Nginx 服务 80 端口，来判断容器是否正常运行
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

##### 构建后验证

```bash
# 构建镜像
$ docker build -t nginx:test .
...
# 启动容器
$ docker run -d --name demo nginx:test
...

# 进入容器内
$ docker exec -it demo bash
# 用户为 root
# 执行 env 命令，出现设置的环境变量
# 当前目录为设置的工作目录
root@*:/usr/share/nginx/html $ env
WELCOME=hello, nginx!

# 查看容器日志;健康检查生效
$ docker logs -f demo
127.0.0.1 - - [07/Jan/2022:09:52:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
```

