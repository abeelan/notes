

## Jmeter 介绍与安装

性能测试：模拟多个用户的操作对服务器硬件性能的影响。

- TPS：每秒事物处理能力
- RT：响应时间



常见性能压力测试工具

- Apache Jmeter：Java 语言开发，开源
- LoadRunner：C 语言开发，商业
- Locust：python 语言开发，开源



Jmeter 优点

- 入门简单，有图形化界面
- 插件机制，可以二次定制
- 支持多操作平台



## 压测脚本录制与编写

#### Jmeter 配置代理

- Test plan -> Add -> Thread Group 

- Thread Group -> Add -> Logic Controller -> Recording Controller
- Test plan -> Add -> Non-Test Elements -> HTTP(S) Test Script Recorder 

<img src="/Users/lan/Library/Application Support/typora-user-images/image-20221024141936125.png" alt="image-20221024141936125" style="zoom:50%;" />

信任后，点击 start。 在 jmeter/bin 目录下，会自动生成证书。Mac 系统双击证书，钥匙串内选择 登录 - 证书，找到 jmeter 证书，设置为始终信任，关闭弹窗，输入电脑密码即可。

<img src="/Users/lan/Library/Application Support/typora-user-images/image-20221024142148018.png" alt="image-20221024142148018" style="zoom:50%;" />



#### 浏览器配置代理

Chrome 浏览器，安装 SwitchOmega 插件，创建 jmeter 情景模式，端口号设置为 8888





#### 录制与回放

开始录制后，访问网页，就会把请求都记录在 Recording Controller 内。

录制的请求比较多，可以通过 HTTP(S) Test Script Recorder 设置过滤：

- URL Patterns to Include 仅录制配置内的域名
- URL Patterns to Exclude 排除配置内的域名（`.*\.(png|gif|js|ttf|woof|css).*`）



录制完成后，关闭录制。添加 View Result Tree，重新 start 录制的请求，即可看到响应信息。



## 虚拟用户并发模拟

Test Group 设置



## 压测结果分析

#### Listener View Result Tree 

查看结果树，直接在当前界面显示请求结果

- RegExp Tester（正则表达式）
- CSS 选择器
- Xpath Tester
- Json Tester



#### Aggregate Report

聚合报告，显示请求时间、tps 等百分比

图形化界面创建好请求后，运行

```shell
# -n 不启动图形化界面
# -t 指定测试脚本
# -l 指定测试结果
# -R 指定远程节点，多个节点中间用逗号隔开
$ ./jmeter.sh -n -t test_script.jmx -l test_script.jtl
```



```shell
# 本地启动一个 python 服务
$ python -m http.server 80
# 本地启动一个 nginx 服务
docker run --name nginx-load-test -p 88:80 -d nginx:1.17.9
```



Backend Listener

将测试数据转入到数据库内



## Jmeter 分布式压测

#### 分布式简介

单机性能瓶颈，CPU、内存、IO等

大型系统， 单机无法模拟，所以需要多台机器对一台服务器施压。



#### 工作节点 - slave 部署

负载机（slaves）：端口 tcp 1099

jmeter/bin 目录下修改配置文件：

- jmeter.properties: 关闭 SSL server.rmi.ssl.disable=true
- system.properties: java.rmi.server.hostname=192.xxx.xxx.xx（本机地址）
- 运行 jmeter-server



等待出现 created remote object... 即可，节点配置完毕，等待控制机连接。



#### 控制节点 - Master 部署

控制端（Master）：端口 udp 4445

jmeter/bin/jmeter.properties 修改配置文件：

- 添加负载机 IP：remote_host = slave 节点的 hostname，多个用逗号隔开
- 关闭 SSL：server.rmi.ssl.disable=true



#### 运行测试

windows 注意关闭防火墙

启动 jmeter 界面程序，运行菜单内，可控制所有远程机器。

- 远程退出所有：节点远程服务直接关闭，远程机器退出命令行
- 如果有多台机器的话，线程数设置为 10，那么就是每台机器上都起 10 个线程。



## 性能监控系统

#### 简介



#### InfluxDB

存储压力测试结果，go 语言开发，8086 端口

```shell
# 新建容器网络
$ docker network create grafana
# 运行容器
$ cd ~/volumes
$ docker run -d --name=influxdb --network grafana -p 8086:8086 -v ${PWD}/influxdb/:/var/lib/influxdb/ influxdb:1.7.10

# 创建数据库
# 方式一
$ curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE jmeter"
# 方式二
$ docker exec -it infulxdb influx
> create database jmeter;
> show databases;
> use jmeter;
> show measurements;
> select * from jmeter limit 3;
```



#### Grafana

读取数据以图形化方式展示测试结果

```shell
$ docker run -d --name grafana --network grafana -p 3000:3000 grafana/grafana:6.6.2
# 默认账号和密码都是 admin
```



配置数据源：设置 -> Data Sources -> Add data source -> influxdb

- URL：http://influxdb:8086，通过容器名加端口号可以访问
- InfluxBD Details: Database -> jmeter
- Min time interval: 配置为 5s，因为 jmeter 每 5s 写入一次，同步一下

测试，出现绿色提示即为成功。

URL 如果不使用容器名的话，可以进入 influxdb 容器内，cat /etc/hosts 找到 IP 地址，填入也行，就是会比较麻烦。



接下来，需要导入一个 dashboard

- Dashboards -> Manage -> Import -> https://grafana.com/grafana/dashboards/5496 -> Load
- 名称可以随意更改
- DB nam 选择 InfluxDB
- Measurement name 填写 jmeter
- 刷新时间为 5 秒，跟前面配置一样
- 点击 Import



#### Jmeter

运行测试生成测试数据，存储到 InfluxDB 内

打开界面

- 创建线程组，添加 http 请求
- 添加监听器 - 后端监听器
- 后端监听器实现修改为 influxdbBackendListener
- InfulxdbUrl 修改为：http://localhost:8086/write?db=jmeter
- application 自定义修改，区分与其他人的数据
- measurement 需要与 jmeter 保持一致
- summaryOnly 修改为 false，会在页面展示错误信息



http 请求内填入本机地址，或者 localhost，发起请求，刷新 grafana 页面，可以看到请求数据。



