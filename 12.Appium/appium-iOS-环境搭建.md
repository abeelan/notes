> 环境搭建时间：2022.3.4
>
> iPhone 12 - 15.1
>
> [参考地址](https://action121.github.io/2019/11/13/Appium-for-Mac-%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.html)
>
> [参考地址2](https://www.gaoyaxuan.net/blog/421.html)



### 准备工作

安装 mac 软件包管理工具 **brew**：https://blog.csdn.net/lan_yangbi/article/details/82774251

依次安装环境依赖的工具。

##### 1、Node

```bash
$ brew install node
$ node -v
v15.6.0
```



##### 2、Xcode

应用商店更新即可，当前版本 Version 12.4 (12D4e)。



##### 3、安装依赖库

```bash
# 苹果设备访问库
$ brew install libimobiledevice --HEAD
# 包安装命令
$ brew install ideviceinstaller

# 安装 iOS 终端调试工具；ios 10+ 使用 appium 需要安装
$ npm install -g ios-deploy

# appium 环境检查工具
$ npm install appium-doctor -g
```



##### 4、Appium Desktop

下载地址：https://github.com/appium/appium-desktop/releases



##### 5、安装 XCUITest Driver

请确保符合以下要求：

- iOS 9.3 +
- Xcode 7 +
- Appium 1.6 +



通过数据线连接 Mac 后，检测连接。

```bash
# 列出设备已安装包列表
$ ideviceinstaller -l

# 检查环境
$ appium-doctor --ios
```



### start 可能出现的问题

多设备未配置 udid

![image-20221204212059341](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221204212059341.png)

提示应用不被认可

![image-20221204212157603](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221204212157603.png)

不能启动 wda

![image-20221204212233719](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221204212233719.png)
