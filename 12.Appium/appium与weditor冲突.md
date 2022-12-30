启动 appium server 后，再次启动 weditor 连接设备报错



```
[E 221216 18:20:07 web:2162] 500 GET /api/v1/devices/android%3Aemulator-5554/screenshot (::1) 2133.77ms
```



解决办法：

- 停掉 appium 服务
- 执行命令：`python3 -m uiautomator2 init` 重装手机上的 agent 服务

