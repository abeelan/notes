> 本文介绍 iOS 设备自动化签到、完成每日任务。



### 环境

- iPhone 15.4
- xcode 12.4
- tidevice 0.6.6
- WebDriverAgent v4.8.4

![基本架构](/Users/lan/Desktop/image-20220907171901337.png)

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



##### 项目代码

**需求**：打开爱维宝贝 APP，自动签到，自动完播广告。

```python
# 部分 api
pkg = "com.zhongwei.aiweibaby"
c = wda.Client('http://localhost:8100')  # 8100为启动WDA设置的端口号

c.session().app_terminate(pkg)  # 杀死应用
c.session().app_activate(pkg)  # 打开应用

c(name="跳过").click_exists(timeout=5)

if c(name="签到得红花").exists:
    c(name="签到得红花").click()
    logger.info("签到成功")
if c(name="今日已签到").exists:
    logger.info("今日已签到")

def watch_video():
    """已经进入校园页面"""
    c(name="去观看").click()

    for i in range(3):
        sleep(10)
        logger.info(f"等待 {10} s")

        x = randint(200, 300)
        y = randint(200, 700)
        c.click(x, y)  # 随机点模拟人工
        
        logger.info(f"click({x}, {y})")
        # TODO：需另起线程监控系统弹窗、跳出APP拉回
        if c.app_current() != pkg:
            c.session().app_activate(pkg)  # 打开应用
    c(name="endcard_close").click_exists(timeout=30)  # 完播关闭
```

