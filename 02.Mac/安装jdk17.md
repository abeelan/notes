下载地址

```
https://www.oracle.com/java/technologies/downloads/#jdk17-mac
```

选择 `x64 DMG Installer` 下载完成后，一路下一步完成安装。



查看当前机器安装的所有 Java 版本

```bash
$ /usr/libexec/java_home -V
17.0.2 (x86_64) "Oracle Corporation" - "Java SE 17.0.2" /Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
    11.0.2 (x86_64) "Oracle Corporation" - "Java SE 11.0.2" /Library/Java/JavaVirtualMachines/jdk-11.0.2.jdk/Contents/Home
    1.8.231.11 (x86_64) "Oracle Corporation" - "Java" /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home  # JRE
    1.8.0_231 (x86_64) "Oracle Corporation" - "Java SE 8" /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home
```



修改 bash_profile 内的 JAVA_HOME 路径

```txt
# JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export PATH
export CLASSPATH
```

