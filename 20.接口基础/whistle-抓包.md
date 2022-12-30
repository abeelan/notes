### 安装启动

https://wproxy.org/whistle/install.html

```bash
$ brew install whistle && w2 start --init
```



安装成功后，打开 Whistle 管理界面 [http://local.whistlejs.com](http://local.whistlejs.com/)



### 命令

```bash
# 启动
$ w2 start
# 加入账号密码启动
$ w2 start -n yourusername -w yourpassword。
# 重启
$ w2 restart
# 停止
$ w2 stop
# 调试模式启动
$ w2 run

# 默认端口 8899，如果被占用，通过 -p 参数指定端口号
```





### 遇到的问题

##### 安卓配置好代理后，实际没走代理

电脑连接了 VPN，有两个 IP 地址。设备 A 连接 VPN IP 可以正常代理请求，设备 B 不行，切换成本地 IP 后成功代理请求。



##### 安卓无法下载 https 证书，下载任务一直等待中

配置好手机代理后，扫描下载证书的二维码，下载任务一直等待中，证书不能下载。

解决办法：

- 更换 chrome 浏览器，别用自带的浏览器
- 访问 http://{IP}:8899/cgi-bin/rootca 下载

参考：https://github.com/avwo/whistle/issues/184
