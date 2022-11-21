> 新建一个 docker slave 节点，并注册到 jenkins 上。
>
> TODO：没有成功，直接在 pipeline 使用 docker 镜像运行



### 下载 docker 插件

- Manage Jenkins：管理
- Manage Plugins：插件管理
- 选中 Available 可用插件栏，搜索 docker
- 下载插件 `Docker`
- 下载插件 docker pipline



### 修改 docker 配置

开放外部连接

```shell
$ vim /lib/systemd/system/docker.service
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375

# 重启服务
$ systemctl daemon-reload
$ systemctl restart docker.service
```



### 配置云

- Manage Jenkins：管理

- configuration System：系统设置
- 页面最底部 Cloud，进入设置云页面
- Docker Cloud details...

![image-20211228153118608](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211228153118608.png)

Docker host uri 是宿主机的 IP 地址，格式要与上图保持一致。



接下来点击 Docker Agent templates 继续配置





###### 记录

apline 配置 ssh：将来放在 dockfile 中构建https://blog.csdn.net/catcher92/article/details/116030171
