> 使用 Nginx 搭建 Web 服务



### Nginx 简介

Nginx (engine x) 是一个高性能的 HTTP 和反向代理 WEB 服务器，通过简单的配置文件即可快速提供服务，性能稳定，系统资源占用少，并发能力强。

- 反向代理：服务器将收到的请求分发到其他服务器
- 负载均衡：流量均匀的分布到后端的服务器上
- HTTP 缓存：缓存服务器，提高用户访问速度



### DEMO

##### 拉取镜像

```shell
$ docker pull nginx:latest
```



##### 启动容器

```shell
$ vim nginx.sh
# -d 在后台运行
# -p 将容器的 80 端口映射到本机的 80 端口
# -v 挂载目录；必须使用绝对路径（/开头的地址）
docker run -d --name nginx -p 80:80 \
	-v /data/nginx/volumes/html:/usr/share/nginx/html \
	-v /data/nginx/volumes/log:/var/log/nginx \
	nginx:latest

$ chmod +x nginx.sh
$ sh nginx.sh
891e09a6bbb5e131f99cd3fe60dfb2687b0939c28ad614d017408ce74478b1a1

# 查看容器状态；status Up
$ docker ps 
```

容器起来后，打开浏览器访问服务器 IP 地址，就可以看到 nginx 的欢迎页！

由于启动命令通过 `-v` 参数挂载了主页面，所以修改本地 index.html 文件可以实时同步到容器内。

```shell
$ echo "<h1>Everythin will be ok.</h1>" > /data/nginx/volumes/html/index.html
```

刷新页面，看看效果。

![image-20211223193910660](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211223193910660.png)



### Nginx 显示目录文件列表

工作中有时会分发安装包、配置文件、测试数据、测试报告等给其他同事，文件上传到服务器发布目录，通过分享链接即可一对多进行文件分发。之前使用 Apache 服务来完成这项工作，正好这几天旧机器回收，借此机会，在新机器上使用 Nginx 搭建该服务，在此记录。



为了安全起见，Nginx 默认不允许展示目录文件列表，通过 Fancy Index 模块可以开启此功能，就像内置的 autoindex 模块一样，同时允许对页面进行定制，会更加美观。

- 自定义页头
- 自定义页脚
- 添加自定义 CSS 样式
- 允许按名称（默认）、修改时间或文件大小对文件进行排序；升序（默认）或降序



登录到服务器，切换为自定义工作目录：`/data1/docker/nginx`，尽量不要放在系统盘，防止系统盘损坏导致数据丢失。



##### 1、下载 [fancyindex](https://www.nginx.com/resources/wiki/modules/fancy_index/ "fancy_index") 

```shell
$ wget -O fancyindex.zip https://github.com/aperezdc/ngx-fancyindex/archive/refs/tags/v0.5.2.zip
$ unzip fancyindex.zip && rm -f nginx-1.21.4.tar.gz
ngx-fancyindex-0.5.2
```



##### 2、下载 [Nginx-Fancyindex-Theme](https://github.com/Naereen/Nginx-Fancyindex-Theme "Nginx-Fancyindex-Theme")

开源的 fancyindex 主题，支持搜索，界面美化，有白色/黑色两种风格。

```shell
$ wget -O fancytheme.zip https://github.com/Naereen/Nginx-Fancyindex-Theme/archive/master.zip
$ unzip fancytheme.zip && rm -f fancytheme.zip
Nginx-Fancyindex-Theme-master
$ mv Nginx-Fancyindex-Theme-master Nginx-Fancyindex-Theme
```



##### 3、下载 Nginx

可以自定义版本，1.21.4 是当前最新版本

```shell
$ wget http://nginx.org/download/nginx-1.21.4.tar.gz
$ tar xvf nginx-1.21.4.tar.gz && rm -f nginx-1.21.4.tar.gz
# 将 fancyindex 放到 nginx 目录内，方便后续编译安装
$ mv ngx-fancyindex-0.5.2/ nginx-1.21.4/model/ngx-fancyindex-0.5.2
```



##### 4、Dockerfile

添加 FancyIndex 模块需要重新编译 Nginx，因此没有使用官方现成的 Nginx 镜像，编写 Dockerfile，基于 apline 镜像重新构建新的镜像。

```dockerfile
# Dockerfile
# apline 面向安全的轻型 Linux 发型版；包管理工具为 apk
FROM alpine:latest
LABEL author="lan"
WORKDIR /root
COPY ./nginx-1.21.4 /usr/local/nginx-1.21.4
# 配置阿里云下载源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories \
    && echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories \
    # 更新 & 安装依赖
    && apk update \
    && apk add --no-cache gcc libc-dev make openssl-dev pcre-dev zlib-dev linux-headers curl \
    && cd /usr/local/nginx-1.21.4/ \
    # 执行编译安装 Nginx
    && ./configure --prefix=/etc/nginx \
        --sbin-path=/usr/sbin/nginx \
        --modules-path=/usr/lib/nginx/modules \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/run/nginx.lock \
        --http-client-body-temp-path=/var/cache/nginx/client_temp \
        --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
        --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
        --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
        --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
        --conf-path=/etc/nginx/nginx.conf \
        --with-http_ssl_module \
        # 指定安装扩展模块路径
        --add-module=/usr/local/nginx-1.21.4/model/ngx-fancyindex-0.5.2 \
        --with-compat \
        --with-file-aio \
        --with-threads \
        --with-http_addition_module \
        --with-http_auth_request_module \
        --with-http_dav_module \
        --with-http_flv_module \
        --with-http_gunzip_module \
        --with-http_gzip_static_module \
        --with-http_mp4_module \
        --with-http_random_index_module \
        --with-http_realip_module \
        --with-http_secure_link_module \
        --with-http_slice_module \
        --with-http_ssl_module \
        --with-http_stub_status_module \
        --with-http_sub_module \
        --with-http_v2_module \
        --with-mail \
        --with-mail_ssl_module \
        --with-stream --with-stream_realip_module \
        --with-stream_ssl_module \
        --with-stream_ssl_preread_module \
    && make \
    && make install \
    && mkdir -p /var/cache/nginx/client_temp \
    && rm -rf /usr/local/nginx-1.21.4 \
    # 修改系统时区为东八区
    && rm -rf /etc/localtime \
    && ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

EXPOSE 80
CMD ["/bin/sh","-c","nginx -g 'daemon off;'"]
```



构建镜像

```shell
# 命令记不住可以直接放到 sh 文件
$ docker build -t nginx-apline:pub -f Dockerfile .
...
Successfully built 93d160ece5d8
Successfully tagged nginx-apline:pub

$ docker images
nginx-apline pub 93d160ece5d8 41 seconds ago 161MB
```



##### 5、编写 nginx.conf 配置文件

```conf
worker_processes  auto;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile      on;
    keepalive_timeout  65;

    fancyindex on;
    fancyindex_localtime on;
    fancyindex_exact_size off;
    fancyindex_header "/Nginx-Fancyindex-Theme-light/header.html";
    fancyindex_footer "/Nginx-Fancyindex-Theme-light/footer.html";
    fancyindex_ignore "examplefile.html";
    fancyindex_ignore "Nginx-Fancyindex-Theme-light";
    fancyindex_name_length 255;

    server {
        # 代理 80 端口；将 location 里面配置的内容反向代理到 80 端口供外部访问
        listen 80;
        server_name localhost;
        # 支持中文显示
        charset utf-8;

        location / {
            autoindex on;
            # 对外访问目录设置
            root /etc/nginx/html;
        }
    }
}
```



##### 6、启动容器

直接封装 sh 脚本，方便执行。

```shell
#!/bin/bash
# 判断容器存在就删除
if [[ -n $(docker ps -q -f "name=^nginx$") ]];
then
    docker rm -f nginx;
fi

# 设置主题；把主题文件放到挂载的主页即可
/bin/cp -rf /data1/docker/nginx/Nginx-Fancyindex-Theme/Nginx-Fancyindex-Theme-light/ /data1/pub/ && \

# 挂载目录
docker run -id --name nginx -p 80:80 \
    -v /data1/pub/:/etc/nginx/html/ \
    -v /data1/docker/nginx/volumes/nginx.conf:/etc/nginx/nginx.conf \
    -v /data1/docker/nginx/volumes/log:/var/log/nginx \
    --restart=always \
    nginx-apline:pub
```



最后附上效果图，页面支持排序、搜索功能，比之前 Apache 的发布目录好多了。

![image-20211230182759926](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211230182759926.png)

