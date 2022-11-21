# K8s

一切皆资源

- pod
- service
- deployment
- ...



集群部署工具：kubeadm（官方）

参考链接：https://www.kubernetes.org.cn/7189.html

- kubectl -> 运行命令，操控集群
- kubelet -> 与集群通信、鉴权、认证等等，一般在 kubeadm 内自动安装
- kubeadm -> 部署工具

机器低于两核会直接报错，部署不起来。



### kubectl

```shell
$ kubectl 动作(get/describe/delete) 资源(pod/deployment/svc) -n 名称空间(不加就跟默认)

# 获取命名空间，默认是 defaule
$ kubectl get namespace
$ kubectl get ns

# 查看日志
$ kubectl log -f

# 登录到容器内
# kubectl exec -it {name} bash

# 创建
# create 如果存在该配置文件会报错，需要先删除再创建
$ kubectl create -f {config 配置文件}
# apply 不需要删除，直接更新，推荐使用
$ kubectl apply -f {config 配置文件}

```







