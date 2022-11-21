> https://github.com/budtmo/docker-android



首先检查服务器是否支持虚拟化技术:https://linux.cn/article-9516-1.html

```bash
$ egrep --color -i "svm|vmx" /proc/cpuinfo
```

如果出现红字就代表机器支持，否则不支持。



