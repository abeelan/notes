App Store 直接下载。



##### 构建应用到真机

可以随意构建应用到模拟器。但是如果要构建到真机上的话，需要苹果的开发者证书。

- 开发者证书：调试真机设备
- 发布证书：将应用提交到 APPStore

设置证书和应用 ID 后，即可进行真机构建。



##### 查看 bundleID 方式

如果报错提示 bundleId 不唯一，则需要在这里修改下，提交应用商店的唯一 ID，类似于安卓的包名。

![image-20221204181049479](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221204181049479.png)



##### 查看应用程序生成的位置

点击 build 后，会构建出一个应用包。包的路径查看方式如图。

注意构建到模拟器和构建到真机的 APP 不通用，构建到真机的包需要签名。

![image-20221204180939825](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221204180939825.png)





