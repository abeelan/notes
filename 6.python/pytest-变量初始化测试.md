记录一个测试类变量初始化问题。



测试场景：初始化变量 `a=1` ，每执行一条测试用例前，都使变量 `a` 自增 1，保证每条用例内的 a 都是不等的。



就像实例变量的值传递一样，如下：

```python
class Demo:
    def __init__(self):
        self.a = 1
        print("init: ", self.a)

    def func_a(self):
        self.a += 1
        print("func a: ", self.a)

    def func_b(self):
        self.a += 1
        print("func b: ", self.a)

if __name__ == '__main__':
    d = Demo()
    d.func_a()
    d.func_b()
    
>>>
init:  1
func a:  2
func b:  3
```



当用 pytest 框架编写测试用例时，提供了两个初始化的方式：

- setup_class：执行测试类时初始化，一个类仅初始化一次
- setup：执行测试方法时初始化，测试类内的每个测试方法执行之前，都会执行



在 setup_class 内初始化变量 `self.a=1`，setup 内给变量 a 自增 1，这样每条测试方法执行前都会去自增 1，思考一下，这样是否能满足需求呢？

```python
class Test:
    def setup_class(self):
        self.a = 1
        print("exec setup class: ", self.a)

    def setup(self):
        self.a += 1
        print("exec setup -", self.a)

    def test_1(self):
        print(self.a)

    def test_2(self):
        print(self.a)

    def test_3(self):
        print(self.a)
```



以上代码的执行结果执行如下，跟我预期不同。



```python
>>>
exec setup class
exec setup - 2
2
exec setup - 2
2
exec setup - 2
2
```



测试方法 1 自增完成后等于 2，执行测试方法 2 的初始化时，变量 a 的值又重新变为 1 自增 1 得到 2，测试方法三同理。



查看 unittest 实现：

```python
class TestDemo(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        cls.a = 1
        print("exec setup class: ", cls.a)

    def setUp(self) -> None:
        self.a += 1
        print("exec setup: ", self.a)

    def test_01(self):
        print("test case 01: ", self.a)

    def test_02(self):
        print("test case 01: ", self.a)
        
>>> 
exec setup class:  1
exec setup:  2
test case 1:  2
exec setup:  2
test case 2:  2
```



与 pytest 执行结果一致。



通过 debug 看下用例执行过程， 执行测试用例 1 的初始化，初始化一个实例对象。

![image-20211206193523593](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211206193523593.png)



执行测试用例 1 时，实例对象不变，所以 a 的值是共用的。

![image-20211206193606075](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211206193606075.png)



执行测试用例 2 的初始化，发现实例变更，所以 a 的初始值并不为 2 ，而是重新创建了一个测试对象后变更为 1。

![image-20211206193630964](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211206193630964.png)



上面这个 demo，踩了个小坑，记录一下，以后别这么用了。为了应对开头的场景，我使用随机数分配给测试用例，问题得到了解决。



