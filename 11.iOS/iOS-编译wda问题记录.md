> 编译 webDriverAgent 遇到的问题记录...

## 获取 UDID 报错

**方法一**

打开 `xcode --> Window --> Divices and Simulators --> identifier`，复制即可

**方法二**

打开 `fidder --> 位置 --> xxx的iPhone --> 信任设备`，点击设备下方可以切换信息，复制 UDID 即可

**xcode：Could not locate device support files.**

⚠️ 上面方法一获取设备 UDID 的时候，遇到该提示

```
Could not locate device support files.
This iPhone XR is running iOS 13.5.1 (17F80), which may not be supported by this version of Xcode.
```

原因是 Xcode 版本过低，不支持调试 13.5.1 的设备，查看当前 xcode 支持版本

```shell
$ cd /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport
$ ls
10.0 10.2 11.0          
...
```

可以 [下载](https://blog.csdn.net/ZuoWeiXiaoDuZuoZuo/article/details/82795731) 手机对应的 `DeviceSupport` 版本，放到该目录下，重启 xcode，再次查看

⚠️ 再次遇到提示

```
dyld_shared_cache_extract_dylibs failed
```

参考 [解决办法](https://www.cnblogs.com/hualuoshuijia/p/10613239.html)

⚠️ 再次遇到提示

```
An error was encountered while enabling development on this device.
Please try rebooting and reconnecting the device. (0xE8000076)
```

网上查了一下，说是因为 xcode 版本过低，手机版本过高，还是得升级 Xcode，郁闷... 先继续往下推进看看。



## 编译 WebDriverAgent 报错

**问题：Messaging unqualified id**

报错信息如下

```
/WebDriverAgent/WebDriverAgentLib/Categories/XCUIElement+FBPickerWheel.m:
Messaging unqualified id
/WebDriverAgent/WebDriverAgentLib/Categories/XCUIElement+FBTyping.m:
Messaging unqualified id
```

**解决**

```shell
# 进入到项目路径下
$ cd ../WebDriverAgent/
$ xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination id={device_udid} GCC_TREAT_WARNINGS_AS_ERRORS=0 test

# 13.5.1 手机版本过高，编译失败
xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination id=00008020-001A08D11A7A002E GCC_TREAT_WARNINGS_AS_ERRORS=0 test

# 12.4.8 成功
xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination id=7017c3493f7a50f2c90a8ec56f1556b92089732c GCC_TREAT_WARNINGS_AS_ERRORS=0 test
```

出现图标即为成功了



### 包名占用

```
The app identifier "com.facebook.WebDriverAgentRunner.lan" cannot be registered to your development team because it is not available. Change your bundle identifier to a unique string to try again.
```

这个 bundle ID 已经被占用了，更换一个。

