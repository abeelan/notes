# [pytest-xdist](https://pypi.org/project/pytest-xdist "官方文档")

> 分布式测试插件

### 安装

```shell
$ pip install pytest-xdist
```

### 测试代码准备
```python
class Test:
    def test_sleep_2(self):
        sleep(2)
        assert False

    def test_sleep_3(self):
        sleep(3)
        assert True
```

### 使用

```shell
# 未使用插件执行，共等待 5 秒
$ pytest test_xdist.py
#  1 failed, 1 passed in 5.22s

# 调起 2 个进程执行任务
$ pytest -n 2
# 1 failed, 1 passed in 3.87s

# 调起与计算机 CPU 内核一样多的进程
$ pytest -n auto
# 1 failed, 1 passed in 4.84s
```
由于启动 CPU 进程也需要时间，所以用例较少的情况下，8 个进程甚至比 2 个进程消耗的时间还要长。用例过多的情况下，系统资源占用过高，导致性能下降。

因此，不推荐使用 `auto` 参数，指定一半的进程数给测试任务就可以了。

### 分发算法配置
- --dist load：默认算法，分发用例给任何可用于执行的进程，不保证顺序；
- --dist loadscope：根据类分组，同一个类下的测试用例都分配给同一个进程运行；
- --dist loadfile：根据 py 文件分组，同一个文件内的用例分配给同一个进程运行，当文件数小于进程数时，启动文件数个进程；
- --dist each：每个进程都会运行此用例，也就是有多少个进程用例就运行多少遍。
- --dist no：不使用并发执行

### 工作原理
1. 控制器在测试开始时根据 -n 参数指定的值产生一个或多个工人，每个工人就是一个迷你的执行器；
2. 工人收集完成的测试用例，将测试集发送会控制器，控制器本身不执行任何收集；
3. 控制器接收所有工人反馈的集合，并执行一些健壮性检查以确保所有工人收集的测试是相同的，如果出现不同，则退出。
4. 如果都相同，那么将测试集合转换为简单索引列表，与原测试集合索引位置相同，这样做可以节省带宽，控制器分配给工人需要执行的索引列表，工人根据列表内 ID 执行测试集合内对应的任务；
5. 如果 --dist each，则向每一个工人都发送完整的测试索引列表；
6. 如果 --dist load，每次分配大约 25% 的索引列表给每个工人，剩余任务等工人执行完成后继续分发；
7. 当工人执行完全部任务时，会将结果返回给控制器，控制器就可以继续使用其他插件，比如报告插件；
8. 全部工作执行完成后，控制器向工人发送关闭信号，进程结束。


### 遇到的问题

```
ERROR collecting：Different tests were collected between gw0 and gw1 
```
原因是参数化测试用例，使用了随机值，导致并发收集测试用例时，`gw0` 和 `gw1` 两个工人收集到的 test id 出现不一致，解决这个问题的办法就是使用固定的参数，即参数生成好后存放起来，再传给参数化装饰器。

在 [GitHub](https://github.com/pytest-dev/pytest-xdist/issues/432 "issues") 上找到了关于这个错误的解释，设计如此，所以修改自己的测试用例吧！



## 如何让scope=session的fixture在test session中仅仅执行一次

pytest-xdist是让每个worker进程执行属于自己的测试用例集下的所有测试用例

这意味着在不同进程中，不同的测试用例可能会调用同一个scope范围级别较高（例如session）的fixture，该fixture则会被执行多次，这不符合scope=session的预期

 

### 如何解决？

虽然pytest-xdist没有内置的支持来确保会话范围的夹具仅执行一次，但是可以通过使用锁定文件进行进程间通信来实现。

 

### 小栗子

1. 下面的示例只需要执行一次login（因为它是只需要执行一次来定义配置选项，等等）
2. 当第一次请求这个fixture时，则会利用FileLock仅产生一次fixture数据
3. 当其他进程再次请求这个fixture时，则会从文件中读取数据



```python
import pytest
from filelock import FileLock


@pytest.fixture(scope="session")
def login():
    print("====登录功能，返回账号，token===")
    with FileLock("session.lock"):
        name = "testyy"
        token = "npoi213bn4"
        # web ui自动化
        # 声明一个driver，再返回

        # 接口自动化
        # 发起一个登录请求，将token返回都可以这样写

    yield name, token
    print("====退出登录！！！====")
```

