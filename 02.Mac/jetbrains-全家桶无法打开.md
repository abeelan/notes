点击程序坞上面的应用程序，跳动两下就没有然后了，无法打开程序；附上解决办法。



前几天改了电脑的用户名称，导致一部分软件都没办法启动；原因是软件的配置文件依然保存原用户名的路径，启动时未获取到目录地址，所以报错了。

---

**以 Pycharm 为例**

找到命令行执行程序：「应用程序」- 右键「显示包内容」- Contents - MacOS - pycharm；

```shell
/Applications/PyCharm.app/Contents/MacOS/pycharm ; exit;

# lan @ tester in ~ [14:58:15] 
$ /Applications/PyCharm.app/Contents/MacOS/pycharm ; exit;
2021-04-07 14:58:15.785 pycharm[55629:2464832] allVms required 1.8*,1.8+
2021-04-07 14:58:15.787 pycharm[55629:2464835] Current Directory: /Users/lan
2021-04-07 14:58:15.787 pycharm[55629:2464835] Value of PYCHARM_VM_OPTIONS is (null)
2021-04-07 14:58:15.787 pycharm[55629:2464835] Processing VMOptions file at /Users/lan/Library/Application Support/JetBrains/PyCharm2020.1/pycharm.vmoptions
2021-04-07 14:58:15.788 pycharm[55629:2464835] Done
Error: could not find libjava.dylib
Failed to GetJREPath()
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
Error opening zip file or JAR manifest missing : /Users/lanzy/.jetbrains/jetbrains-agent-v3.2.1.4255.9c8
Error occurred during initialization of VM
agent library failed to init: instrument

[进程已完成]
```

日志显示获取 jre 环境失败，关键信息就是这句，这个路径是老用户名，需要修改为最新：

`Error opening zip file or JAR manifest missing : /Users/lanzy/.jetbrains/jetbrains-agent-v3.2.1.4255.9c8`

```shell
$ cd /Users/lan/Library/Application Support/JetBrains/PyCharm2020.1/
$ subl pycharm.vmoptions
# 将 `javaagent` 这一项改为新用户名即可
...
-javaagent:/Users/lan/.jetbrains/jetbrains-agent-v3.2.1.4255.9c8=51aaea47
```



再次启动 Pycharm，正常使用～



---

idea 也是同样的问题，修改 idea.vmoptions 文件下的 javaagent 配置即可。