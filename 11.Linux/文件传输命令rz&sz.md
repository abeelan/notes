> 软件版本：secureCRT Version 8.7.3 (build 2279)



### 安装

```shell
# centOS
$ yum -y install lrzsz
...
lrzsz.x86_64 0:0.12.20-36.el7
Complete!
```



### 使用

rz: Receive Zmodem

sz: Send Zmodem

rz 和 sz 都是使用 [Zmodem 文件传输协议](https://zh.wikipedia.org/wiki/ZMODEM)



## sz

从服务器传输文件到电脑，对于服务器来说，是发送（send），所以 sz

```shell
# 单个文件
$ sz {filename}  
# 多个文件
$ sz {filename1} {filename2} 
# 目录下文件
$ sz {dirname}/*
```

文件下载默认路径：`~/Document`

设置自定义路径：`secureCRT --> Preferences --> General --> Default Session --> Edit Default Settings --> X/Y/Zmodem`



## rz

从电脑上传文件到服务器，对于服务器来说，是接收（receive），所以 rz

```shell
# -be 解决文件乱码 
$ rz -be  # 弹框内选择要上传的文件
```

