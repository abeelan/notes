让 iOS 拥有像安卓 adb 一样的命令行工具。

<!--more-->

## 需求

IOS 手机也能像安卓一样，做一个桌面工具，支持：

- 截图
- 获取手机信息
- App 安装，启动，停止，查看
- 获取应用信息
- 指定应用性能采集（类似 PerfDog）
- 其他

最重要的是支持 Windows、Linux、Mac 跨平台使用。



## 工具确定

- tidevice

感谢 [codebluesky 发帖](https://testerhome.com/topics/27758) & [taobao-iphone-device](https://github.com/alibaba/taobao-iphone-device)



## 环境准备

- 电脑（先用 Mac 测试）
- Python 3.7 +（3.7.3 测试）
- 确保手机上安装有 [WebDriverAgent](https://github.com/appium/WebDriverAgent) 应用



### 安装

#### 1. 安装 tidevice

```shell
$ pip3 install -U tidevice
...
Successfully installed colored-1.4.2 simple-tornado-0.2.0 tidevice-0.1.11
```



#### 2. 安装 WebDriverAgent

[教程](https://mp.weixin.qq.com/s/48zsJvSSPiQBOmVxL8KY0g)

编译相关问题参考 >>> 桌面文档 - WebDriverAgent编译遇到的问题.md



## tidevice 命令行使用

查看版本号

```shell
$ tidevice version
tidevice version 0.1.11
```

列出连接设备

```shell
$ tidevice list
List of apple devices attached

# json 格式列出设备信息(uid&name)
$ tidevice list --json
```

应用管理

```shell
# 安装应用
$ tidevice install example.ipa

# 指定设备安装
$ tidevice --udid $UDID install https://example.org/example.ipa

# 卸载应用
$ tidevice uninstall com.example.demo

# 启动应用
$ tidevice launch com.example.demo

# 停止应用
$ tidevice kill com.example.demo

# 查看已安装应用
$ tidevice applist
```



