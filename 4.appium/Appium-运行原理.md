### 简介

[Appium](https://appium.io/docs/cn/about-appium/intro/ "Appium docs") 是一个移动端自动化测试框架，可用于测试：

- 原生应用：安卓或 iOS 应用
- 移动网页应用：网页应用，h5，safari 或者手机 chrome
- 混合应用：原生应用嵌套 webview

支持跨平台，底层多引擎可切换，生态丰富，社区强大。



### 概念

**客户端 / 服务器 架构**
`Appium` 的核心一个是暴露 `REST API` 的 `WEB` 服务器。它接受来自客户端的连接，监听命令并发送指令到移动设备上执行，返回 `HTTP` 响应来描述执行结果。



[**Appium 客户端**](https://appium.io/docs/cn/about-appium/appium-clients/index.html "客户端程序库列表")
客户端程序库，各种语言实现的 `Appium-client API`，是 `Appium` 对 `WebDriver` 协议的扩展。这些库封装了标准的 `Selenium` 客户端，提供了所有 [JSON Wire protocol](https://w3c.github.io/webdriver/webdriver-spec.html) 指定的常规 selenium 命令，并额外添加操控移动设备相关的命令，例如 **多点触控手势** 和 **屏幕方向**。



**Appium 服务器**
Appium 是一个用 `Node.js` 写的服务器，可以放在本机，也可以放在云端。主要负责两件事情：

- 默认开启 `4723` 端口，监听来自客户端的 `HTTP` 请求

- 本地开启 `4724` 端口，将请求转发给移动端



[**预期能力（Desired Capabilities）**](https://appium.io/docs/cn/writing-running-appium/caps/index.html "预期能力汇总") 
它告诉服务器我们想要启动什么类型的自动化会话，本质上是一个 `key-value` 形式的对象。脚本通过请求以 `json` 格式发送测试设备信息给服务端，服务端来完成该类型会话的创建。



**会话（Session）**
自动化测试始终在一个会话的上下文中执行，客户端程序库以各自的方式发起与服务器的会话，但最终都会发给服务器一个 `POST /session` 请求，请求中包含一个被称作「预期能力」的 JSON 对象。这时服务器就会开启这个自动化会话，并返回一个用于发送后续命令的会话 ID。

Session 对象存储特定用户会话所需的属性及配置信息，对应到这里其实就是 `Desired Capabilities` 中的配置信息参数。



[**Appium Desktop**](https://github.com/appium/appium-desktop "Appium Desktop")

Appium 界面版本，打包了 Appium 服务器运行需要的所有东西。提供 `Inspector` 查看应用程序的层级结构。



各平台测试引擎列表：

- iOS 9.3 及以上: 苹果的 [**XCUITest**](https://developer.apple.com/reference/xctest)
- iOS 9.3 及以下: 苹果的 [UIAutomation](https://web.archive.org/web/20160904214108/https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/)
- Android 4.3+: 谷歌的 [UiAutomator / **UiAutomator2**](https://developer.android.com/training/testing/ui-automator)
- Android 2.3+: 谷歌的 [Instrumentation](http://developer.android.com/reference/android/app/Instrumentation.html)
- Windows: 微软的 [WinAppDriver](http://github.com/microsoft/winappdriver)



### 框架介绍

客户端发送请求到服务端，服务端将请求转为可执行指令发送到设备，执行操作后返回结果。

<img src="https://secure2.wostatic.cn/static/rnioczxzYK4oun1UmCa1ws/image.png?auth_key=1668346066-sX9ayRa3erCqXv4By5JEmL-0-a1fe5050fd01fc095474bce389721cbf" alt="img" style="zoom:50%;" />

**以 Android 端为例**

- 启动 `Appium Server`，监听 `4723` 端口的请求；
- 运行代码，`client` 向 `server` 发送带有设备信息的 `HTTP` 请求，`server` 监听到该设备信息；
- 初始化移动设备，安装 `bootstrap.jar` 并自动开启设备的 `4724` 端口；
- `server` 开启 `4724` 端口用于和 `bootstrap.jar` 通信，等待客户端连接；
- `server` 根据监听到的 `client` 请求生成对应的自动化会话，并返回 `SessionID` 给 `client`；
- `client` 将脚本转化为请求并携带 `SessionID` 发送给 `server`；
- `server` 将请求解析后，通过 `4724` 端口转发给 `bootstrap.jar`，这时的 `Appium server` 转变为客户端；
- `bootstrap.jar` 将请求信息转化为 `uiautomator` 指令，让 `uiautomator` 进行处理并执行;
- 执行后的请求响应原路返回给脚本，脚本再进行下一次的请求；
- 代码执行 `quit` 后，关闭 `session` 和进程，测试结束。



Appium Server 作为服务端，同时也是客户端，是一个双向通信连接实现数据交换的过程。

> 总体来说，Appium Server 既扮演着 server 的角色又扮演着 client 角色。
>
> 当它面对代码脚本的时候是一个 server，接收脚本发送的请求指令；
>
> 当它面对在终端设备上部署的原生代理（Android -> bootstrap.jar | iOS -> wda）时又是一个client，把接收到的网络指令以规定的形式发出去，不同平台的代理将请求解析为原生语言指令，通过原生测试框架操作 APP，这就是 appium 实现跨平台的方式。



### 环境安装

- java 1.8+
- Android SDK
- Node. js (>=10)、npm(>=6)
- python3
- appium-desktop（非必须）
- appium-python-client

`Appium Server` 是 `node.js` 写的，必须先安装 [node](https://blog.csdn.net/lan_yangbi/article/details/106569174 "node安装")

```bash
# 安装 appium
$ npm install -g appium
$ appium

# 安装 python client
$ pip install Appium-Python-Client
```

demo

```Python
# 验证环境是否正常
from appium import webdriver

desired_caps = {
    'platformName': 'Android',
    'platformVersion': '10.0',
    'deviceName': 'Android Emulator',
    'appPackage': "com.android.settings",
    "appActivity": ".HWSettings"
}

driver = webdriver.Remote("http://localhost:4723/wd/hub", desired_caps)
driver.quit()
```



### dom 结构

- dom：`Document Object Model`，文档对象模型
- dom 应用：用于表示界面的控件层级，界面的结构化描述。常见的格式为 `html\xml`。核心元素为节点和属性。



Andoid 应用的层级结构和 `html` 不一样，是一个定制的 `xml`。`App source` 类似于 dom，表示界面所有控件的结构。每个控件都有它的属性，但没有 css 属性。



IOS 与 Android 的 dom 结构类似，但是属性有所区别，例：

- Android resourceid = iOS name
- Android content-des = IOS accessibility-id

