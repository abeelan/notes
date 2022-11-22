安装

```shell
$ yum -y install rsync
```



同机器同步

```shell
# 增量同步
$ rsync -avz folder1/ folder2/
```



远程同步

```shell
# 服务端
$ rsync -avz /data3/a.py qard@10.26.31.141:/data3/lan/

# 从远程同步文件到本地
$ rsync -avz -e 'ssh -p 22' root@远程IP:/root/hello /root

# 远程服务 需要 安装 rsync 服务
```



```
motd file -> motd文件位置
log file -> 日志文件位置
path -> 默认路径位置
use chroot -> 是否限定在该目录下，默认为true，当有软连接时，需要改为fasle,如果为true就限定为模块默认目录
read only -> 只读配置（yes or no）
list=true -> 是否可以列出模块名
uid = root -> 传输使用的用户名
gid = root -> 传输使用的用户组
auth users -> 认证用户名
secrets file=/etc/rsyncd.passwd -> 指定密码文件，如果设定验证用户，这一项必须设置，设定密码权限为400,密码文件/etc/rsyncd.passwd的内容格式为：username:password
hosts allow=192.168.0.101  -> 设置可以允许访问的主机，可以是网段，多个Ip地址用空格隔开
hosts deny 禁止的主机，host的两项可以使用*表任意。
```



