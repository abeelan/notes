原理图

![image-20221204204246377](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221204204246377.png)



### 环境

- iPhone 15.4
- xcode 12.4
- tidevice 0.6.6
- WebDriverAgent v4.8.4



初始化工作

```shell
# 安装 tidevice
$ pip install -U tidevice

# 安装 python wda client
$ pip install -U facebook-wda

# clone wda 源码
$ cd workspace
$ git clone https://github.com/appium/WebDriverAgent.git
$ cd WebDriverAgent
# 目前的版本不需要执行 ./Scripts/bootstrap.sh
# 双击打开 WebDriverAgent.xcodeproj 项目
```

编译安装 `APP` 请参考： IOS测试 | facebook-wda 环境搭建篇



出现如下报错信息需要信任开发者，点击「通用 - VPN与设备管理 - 信任开发者」即可。

> The operation couldn’t be completed.  Unable to launch com.facebook.WebDriverAgentRunner.lan.xctrunner because it has an invalid code signature, inadequate entitlements or its profile has not been explicitly trusted by the user.



通过 `tidevice` 启动 `wda`：

```shell
$ tidevice -u [udid] wdaproxy -B [wda bundle Id] --port 8100
...
WebDriverAgent start successfully
```



### DEMO

##### 抓取页面元素

注意，在当前版本中，`/inspector` 接口已经移除，不能使用。

> mykola-mokhnach: The built-in inspector module has been removed from the fork in favour of Appium Desktop's one
>
>
> 内置的检查器模块已经从 fork 中移除，取而代之的是 Appium Desktop。

不想下载 `appium Desktop`，这里使用源码方式获取元素属性。

```python
c.source()  # xml 格式
c.source(accessible=True)  # json 格式
```

在 `dom` 结构内找到预期元素的属性，进行定位。

