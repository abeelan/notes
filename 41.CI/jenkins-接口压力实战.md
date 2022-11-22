> 接口压力测试实战，集成到 jenkins 项目。



创建一个自由风格的新项目。

配置 git 项目上拉取代码，git 不稳定，直接拷贝项目到 workspace/weather_stress 目录下

配置 build -> execute shell

```shell
$ cd iInterface_python/jmx
$ sh auto_stress_test.sh
```

下载 Groovy Postbuild 插件，目的是解除 jenkins 对于 js 渲染的限制。Groovy Script 添加如下代码。

```shell
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP","")
```

增加 publish html report 用于展示报告，增加三个，分别对应 10、20、30

![image-20210829160515927](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210829160515927.png)

---



### 安装 jmeter 环境

```shell
$ cd iInterface_python/jmx
$ sh auto_stress_test.sh
auto_stress_test.sh: 15: auto_stress_test.sh: Syntax error: "(" unexpected
# 出现报错
# 代码对于标准bash而言没有错，Debian为了加快开机速度，用dash代替了传统的bash
$ sudo dpkg-reconfigure dash
# 输入 n，继续运行

# 安装 jmeter
# $ wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.4.1.tgz
# 下载的太慢了，直接从宿主机拷贝过去
$ docker cp Downloads/jmeter-5.3.zip jenkins:/opt
$ cd /opt && unzip jmeter-5.3.zip

# 修改 auto_stess_test.sh
# 需要在系统变量中定义jmeter根目录的位置，如下
# export jmeter_path="/your jmeter path/"
# 放开上面的注释
export jmeter_path="/opt/jmeter-5.3/"
```



构建，即可搞定。



接口性能参考标准（百分之90，参考，十几年前的标准）：

- 2秒：良好
- 8秒：可忍受
- 12秒：不可忍受



吞吐量

- 随着流量增大，TPS 会显著增大，继续增加流量，TPS 会呈现下滑趋势，这个拐点就是性能瓶颈。
- 可以根据每天的访问量去换算（TPS 12 的话，大概一天可以承受一百万的访问量）



也需要监控 server 物理的值，比如 CPU、内存、网络等等。





