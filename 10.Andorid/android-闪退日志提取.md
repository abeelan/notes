安卓手机测试过程中，遇到应用闪退，通常需要在 `BUG` 信息上附加崩溃日志，便于开发同学快速定位问题。



存储日志命令为：
`adb logcat -v time > crash_log.log`

这个方法抓到的日志过多，很多都是无用的系统日志或级别过低的打印日志，所以需要做一次过滤，`logcat` 可以根据日志等级过滤。将日志级别设置为 `ERROR`，就可以过滤掉一大部分无用信息。

命令修改为：
`adb logcat -v time *:E > crash_log.log`

> **备注：**
>
> - 星号 `*` 代表日志任意 `tag` 都只输出 `Error` 级别以上日志；
> - MacOS 命令行直接使用 `*:E` 会报错 `no matches found: *:E`，所以需要添加双引号或转义符来解决：`"*:E"` 或 `\*:E`

获取到日志后，会发现，还是存在一部分系统的错误日志，如何再次过滤呢？

目标：仅需要某个闪退的全部堆栈信息

思路：

1. 通过命令行做第一次过滤，获取到 `ERROR` 级别的原始日志；
2. 逐行读取日志，如果查找到关键字 `FATAL EXCEPTION: main` 则代表出现一次闪退，记录该进程 `PID` 存储到列表内；
3. 判断当 `PID` 列表非空时，创建日志文件，将该 `PID` 的所有日志写入到新文件内。

这样就能满足需求了，接下来代码实现，验证效果。



##  获取日志

人工操作步骤为：命令行输入命令，获取日志后，`Ctrl+C` 关闭日志

代码也一样，实现方法：

- 模拟命令行：`subprocess`模块
- 等待日志获取：安卓日志缓冲区一般为`64KB`，可以通过 `time.sleep()` 暂停程序 `5` 秒（可以自定义时间，我是感觉够用了）；
- `Ctrl+C`：通过命令杀死 `adb logcat` 进程

```
def log_cat(file_path, device=""):
    """获取 adb 日志
    """
    command = rf"adb {device} logcat -v time \*:E > {file_path}"
    subprocess.Popen(command, shell=True)
    time.sleep(5)
    try:
        if platform.system() == "Darwin" or "Linux":
            subprocess.call(
                "kill -9 $(ps -ef | grep 'adb logcat' | sed -n 1p | awk '{print $2}')",
                shell=True
            )
        else:
            # Windows 不知道能否生效，网上查的这个方法，没有机器测试
            subprocess.Popen("taskkill /F /IM adb.exe")
    except Exception as e:
        logging.warning(f"Kill「adb logcat」called Exception，message：{e}")
```



## 统计出现闪退次数

关键字：闪退日志每次出现都会带一行关键字 `FATAL EXCEPTION: main`，`FATAL` 是日志级别，比 `ERROR` 更高一级，类似 Python 中的 `critical`，无法使用的状态，所以当遇到这一行日志时，我就认为出现闪退了；

第二步的实现思路为：逐行读取保存的日志文件，出现关键字就获取当前进程号，并添加到列表中，最后统计列表长度即可。

```
def get_crash_pid_list(file_path):
    """读取日志，获取出现关键字（crash）的次数
    """
    keyword = "FATAL EXCEPTION: main"
    crash_pid = []

    with open(file_path, encoding="utf-8") as f:
        for line in f.readlines():
            if keyword in line:
                # 提取出日志行内所有数字（日期 + PID）
                data = re.findall(r"\d+", line)  
                pid = data[-1]
                crash_pid.append(pid)
    logging.info(f"Crash PID list >>> {crash_pid}")
    return crash_pid
```



## 转储闪退日志

这里需要进行一次判断：

- 如果不存在闪退，直接给出提示，退出程序
- 如果存在，创建新的日志文件，写入设备信息，根据进程号提取日志

**获取设备信息**
日志提交后，被开发同学问过几次机型和系统，索性就直接加到日志里面吧

```
def get_device_info(device=""):
    """获取设备信息
    """
    model = os.popen(f"adb {device} shell getprop ro.product.model").read().strip()
    manufacturer = os.popen(f"adb {device} shell getprop ro.product.manufacturer").read().strip()

    version = os.popen(f"adb {device} shell getprop ro.build.version.release").read().strip()
    sdk_version = os.popen(f"adb {device} shell getprop ro.build.version.sdk").read().strip()

    output = f"DeviceInfo: {manufacturer} {model} | Android {version} (API {sdk_version})"
    return output
```

**转存日志**
逐行读取，匹配进程号，写入到新的日志文件内，如果不填，则走默认路径

```
def dump_crash_log(file_path="", dump_path="", device=""):
    """转储带有 crash pid 的日志
    """
    # 设置默认转储路径
    if not file_path:
        file_path = "logcat.log"
    if not dump_path:
        dump_path = "logcat_crash.log"
    if device:
        device = f"-s {device}"

    log_cat(file_path, device)
    device_info = get_device_info(device)
    pid_list = get_crash_pid_list(file_path)

    if pid_list:
        # 创建转储日志并写入
        with open(dump_path, "w+", encoding="utf-8") as f:  
            f.write(f"{'-' * 50}\n")
            f.write(f"{device_info}\n共出现 {len(pid_list)} 次闪退\n")
            f.write(f"{'-' * 50}\n")
            # 读取原始日志
            with open(file_path, encoding="utf-8") as f1:
                for line in f1.readlines():
                    for pid in pid_list:
                        if pid in line:
                            if "FATAL" in line:
                                f.write("\n# begging of crash --- >>>\n")
                            f.write(line)
        logging.info(f"Crash log path: {dump_path}")
    else:
        logging.info(f"Not found 'FATAL EXCEPTION: main' in {file_path}")

if __name__ == '__main__':
    dump_crash_log()
```

------

最后执行效果如下：

**控制台输出**

```
[ I 201231 13:57:05 dump_crash_log:44 ] PID LIST >>> ['26997', '28084', '6964']
[ I 201231 13:57:05 dump_crash_log:88 ] Crash log path: logcat_crash.log
```

**目录结构**

- dump_crash_log.py
- logcat.log
- logcat_crash.log

**日志内容**

```
--------------------------------------------------
DeviceInfo: HUAWEI ELE-AL00 | Android 10 (API 29)
共出现 3 次闪退
--------------------------------------------------

# begging of crash --- >>>
11-02 17:39:58.802 E/AndroidRuntime(26997): FATAL EXCEPTION: main
11-02 17:39:58.802 E/AndroidRuntime(26997): Process: com.esbook.reader, PID: 26997
11-02 17:39:58.802 E/AndroidRuntime(26997): java.lang.IllegalArgumentException: 这是异常信息
11-02 17:39:58.802 E/AndroidRuntime(26997):     at 这是堆栈信息
11-02 17:39:58.802 E/AndroidRuntime(26997):     at 这是堆栈信息
...

# begging of crash --- >>>
11-02 17:55:39.306 E/AndroidRuntime(28084): FATAL EXCEPTION: main
11-02 17:55:39.306 E/AndroidRuntime(28084): Process: com.esbook.reader, PID: 28084
11-02 17:55:39.306 E/AndroidRuntime(28084): java.lang.NullPointerException: 这是异常信息
11-02 17:55:39.306 E/AndroidRuntime(28084):     at 这是堆栈信息
11-02 17:55:39.306 E/AndroidRuntime(28084):     at 这是堆栈信息
...

# begging of crash --- >>>
11-02 18:15:37.625 E/AndroidRuntime( 6964): FATAL EXCEPTION: main
11-02 18:15:37.625 E/AndroidRuntime( 6964): Process: com.esbook.reader, PID: 6964
11-02 18:15:37.625 E/AndroidRuntime( 6964):     at 这是堆栈信息
11-02 18:15:37.625 E/AndroidRuntime( 6964):     at 这是堆栈信息
...
```

---

大功告成，这样提取日志就会方便许多啦！😄