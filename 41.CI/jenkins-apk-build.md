# 测试 APP 构建、打包过程

### 准备

安卓源码

```shell
# https://github.com/princeqjzh/AndroidSampleApp.git
$ git clone git@github.com:princeqjzh/AndroidSampleApp.git
```

安卓打包命令

```shell
# clean 清空 build 目录下的构建文件
# assemble 打包；比如 assembleRealese 或者渠道包
$ gradlew clean assembleDebug

# APK 输出路径
# /app/build/outputs/apk/debug/app-debug.apk
```

打包环境要求

- JDK
- Android SDK
- Gradle

编译完成后安装包

```shell
$ adb install app-debug.apk
Success
```



### 总结

配置 job 1 用于构建 APP；

配置 job 2 用于构建 自动化测试（子节点 - 模拟器运行）；

父子关联（job 1 添加稳定构建后执行 job 2）



### 遇到的报错

```
# 执行编译命令时出现报错
Required by:
project :
> Could not resolve com.android.tools.build:gradle:3.4.1.
> Could not get resource 'https://dl.google.com/dl/android/maven2/com/android/tools/build/gradle/3.4.1/gradle-3.4.1.pom'.
> Could not GET 'https://dl.google.com/dl/android/maven2/com/android/tools/build/gradle/3.4.1/gradle-3.4.1.pom'.
> Connect to dl.google.com:443 [dl.google.com/203.208.43.97] failed: Read timed out
```

显示无法请求 dl.google.com，本地 ping 这个域名网络链接是正常的呀，不知道什么原因。

一顿折腾都没用，就在要放弃的时候，再次执行了一下，好了。。。



`BUILD SUCCESSFUL in 1m 37s`

