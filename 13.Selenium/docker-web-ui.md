> 部署 selenium web 自动化测试实战。



参考资料：[Selenium Grid使用](https://blog.csdn.net/lb245557472/article/details/91966770)



### 安装 chrome 和 chromedriver

https://blog.csdn.net/GO_D_OG/article/details/79073727

需要使用无头浏览器运行，不然报错。



## 搭建分布式 UI 自动化测试环境

自动化测试发展从刚开始的单机单线程到单机多线程再到多机分布式操作。

UI 自动化测试通过 Grid hub 分发用例到不同的节点，执行用例。

grid 负载均衡是通过查看不同节点启动的浏览器数量，如果数量多证明繁忙，会优先分发到浏览器数量少的node上

github 搜索 docker- selenium

standalone 镜像内包含 hub 和 node，一般来做调试用，不能做分布式。



### 启动 hub

```shell
$ docker run --name=hub -p 5001:4444 -e GRID_TIMEOUT=0 -e GRID_THROW_ON_CAPABILITY_NOT_PRESENT=true -e GRID_NEW_SESSION_WAIT_TIMEOUT=-1 -e GRID_BROWSER_TIMEOUT=15000 -e GRID_TIMEOUT=30000 -e GRID_CLEAN_UP_CYCLE=30000 -d selenium/hub:3.7.1-beryllium
```

访问：http://localhost:5001/，得到下图中的页面。

![image-20210825120833235](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210825120833235.png)

### 启动 node

```shell
$ docker run --name=chrome -p 5902:5900 -e NODE_MAX_INSTANCES=6 -e NODE_MAX_SESSION=6 -e NODE_REGISTER_CYCLE=5000 -e DBUS_SESSION_BUS_ADDRESS=/dev/null -v /dev/shm:/dev/shm --link hub -d selenium/node-chrome-debug:3.7.1-beryllium
```

由于 node 节点运行的是远程浏览器，selenium/node-chrome-debug 镜像，提供远程 debug 的功能，通过 VNC 远程桌面服务来进行查看浏览器的运行状态，所以 node 也需要映射一个端口来与 VNC 进行通信。VNC 密码是 secret。

NODE_MAX_INSTANCES、NODE_MAX_SESSION 这两个参数决定该 node 节点启动浏览器的上限，启动浏览器是非常消耗 CPU 的，如果启动过多，可能会压垮机器，需要设置上限进行保护。

-v /dev/shm:/dev/shm：这个参数是优化内存使用的，如果没有会因为内存问题容易崩溃。

--link hub：把 hub 的网络信息发送给 node 节点，将 hub 链接到容器上

访问：http://localhost:5001/，点击 console，跳转到下图页面。

![image-20210825120912920](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210825120912920.png)

通过该命令可以启动多个 node，注意更换下名称和端口号即可，刷新当前页面，会看到有多个节点。

node 也可以启动不同的浏览器，用来做浏览器的兼容性测试。



### 演示

环境已经搭建完成了， 那么接下来通过代码来看看实际效果。

```python
# demo.py
import selenium.webdriver.remote.webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
from time import sleep

def test_baidu_search():
    driver = selenium.webdriver.remote.webdriver.WebDriver(
        command_executor="http://127.0.0.1:5001/wd/hub",
        desired_capabilities=DesiredCapabilities.CHROME
    )
    driver.get("http://www.baidu.com")
    driver.find_element_by_id("kw").send_keys("python")
    driver.find_element_by_id("su").click()
    sleep(2)
    driver.quit()
```

通过 remote 将远程地址修改为 hub 的地址，由于是本地服务，所以只需要修改一下端口即可。

运行测试用例，发现执行通过，由于是运行在容器内，本地是看不到浏览器窗口的，还需要通过 VNC Viewer 来查看远程浏览器。

> VNC Viewer
>
> 官网下载：https://www.realvnc.com/en/connect/download/viewer/
>
> 打开应用后，command + N 新建远程链接，填写远程 IP 和端口，名称随意。

server 处输入 {本机IP}:5902，连接上面的 node-chrome-debug，密码为 secret；

链接成功后，执行测试用例，即可看到浏览器被拉起，搜索关键词 python 后点击搜索。
