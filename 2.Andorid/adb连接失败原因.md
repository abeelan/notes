> adb 连接不上设备的解决办法。

首先确保电脑上的`android-platform-tools`版本与设备的`API`版本匹配。
```shell
$ adb --version
Android Debug Bridge version 1.0.40
Version 4986621
Installed as /usr/local/Caskroom/android-platform-tools/28.0.1/platform-tools/adb
```

比如上面，我电脑的`adb`版本是`28`，连接一个`10.0`系统(`API 29`)的设备，死活连接不上。。。

所以当遇到设备无法连接，先检查或尝试更新`adb`版本，确保版本可以匹配。

版本匹配参考 [Android SDK](https://developer.android.com/guide/topics/manifest/uses-sdk-element "Android SDK")

```shell
# brew 更新 adb 命令
$ brew upgrade android-platform-tools
```

如果还是连接不上，通常的解决办法：

- 重新插拔数据线；
- 重新关开**开发者选项**，插拔数据线；
- 重新关开 **USB 调试**，插拔数据线；
- **撤销** USB 调试授权，插拔数据线。



#### 不显示设备

已通过数据线连接手机和电脑，但是不显示设备号。

```shell
$ adb devices
List of devices attached
* daemon not running; starting now at tcp:5037
* daemon started successfully
```

遇到这种情况，先检查设备是否真正的连接到电脑上了，执行如下命令。

```shell
$ system_profiler SPUSBDataType
...
ELE-AL00:
	Product ID: 0x107e
	Vendor ID: 0x12d1  (Huawei Technologies Co., Ltd.)
	...
```

当执行完命令，没有在输出内找到设备信息，那就**更换 USB 数据线**重试吧。



记录设备的`Vendor ID`并添加到`adb_usb.ini`文件内，如果没有该文件就新建一个。

```shell
$ vim /Users/用户名/.android/adb_usb.ini
```

插拔设备重试。



#### 未授权

显示连接设备，但是设备处于`unauthorized`状态，且手机不弹授权的提示框。

```shell
$ adb devices
GBG5T19731003744	unauthorized
```



不弹授权的弹框，可能是电脑上有过这款手机的历史授权记录，即使授权失败了，也不会再弹窗。

```shell
$ cd /Users/用户名/.android/
$ rm adbkey adb_usb.ini
```

删除上面两个配置文件，插拔设备重试。