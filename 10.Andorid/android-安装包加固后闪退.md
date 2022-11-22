这版本测试时，出现装包后启动 CRASH 的情况（仅在部分机型上面出现），出现该问题后，APP 一直无法正常启动，重新安装后，启动正常；尝试定位问题。



**闪退日志：**

```log
java.lang.UnsatisfiedLinkError:dlopenfailed: library "/data/data/cn.wejuan.reader/.jiagu/libjiagu.so" not found
```



**问题排查：**

1. 看日志应该是加固的问题， 启动时文件创建失败导致未找到；

2. 没有任何 APP 相关的报错信息；

3. 非必现。



**BUG 复现步骤：**

1. 手动重复安装卸载；未复现
2. 通过命令快速重复安装后卸载；尝试 20 次左右复现了

**闪退日志与之前一样，依然无法具体定位问题**

---

开发重新打包后未加固，测试快速安装卸载 20 次，执行 5 遍，未复现；

更换加固包，测试快速安装卸载 20 次复现；

问题基本定位，与加固有关？

问题反馈至第三方，第三方承认有问题，等待解决。

---

贴上快速安装卸载脚本，待问题解决，验 BUG 用。

```shell
#!/bin/zsh
# 该脚本为了复现 BUG：某些机型启动出现 CRASH，报错日志为加固问题
# 在小米6上复现方法：快速安装启动
# 循环
# 1. 安装 apk；
# 2. 启动 APP；
# 3. 点击弹框内确定进入APP；【注释该行：经测试，与是否进入APP，关系不大】
# 4. 卸载 APP
echo "Usage : ./resart.sh {apk_path} {loop_count}"
echo "Package Name: $1"echo "Loop Count  : $2"
count=$2  # 循环次数

# 协议弹窗确定按钮 x y 坐标
agreement_box_x=727  
agreement_box_y=1303

# 系统权限弹窗 x y 坐标
box_x=800  
box_y=1800

for ((i=1; i<=count; i++))
do
adb install $1
adb shell am start -n "cn.wejuan.reader/com.esbook.reader.activity.ActLoading"
sleep 2

# adb shell input tap $agreement_box_x $agreement_box_y
# sleep 0.5
# adb shell input tap $box_x $box_y
# sleep 0.5
# adb shell input tap $box_x $box_y
# sleep 3

adb uninstall cn.wejuan.reader
echo "========= Loop count >>> $i ============"
done
```





