### 通过 python 调用

```bash
$ python -m pytest [...]
```

等同于直接调用 pytest，通过 python 调用会将当前目录添加到 sys.path



退出码

- Exit code 0: 找到所有测试用例并测试通过
- Exit code 1: 部分用例运行失败
- Exit code 2: 用户中断了测试
- Exit code 3: 执行过程中发生了内部错误
- Exit code 4: pytest 命令行使用错误
- Exit code 5: 没有找到任何测试用例



### 命令行参数

```bash
# 显示 pytest 的 import 的路径
$ pytest --version
# 显示所有 fixtures
$ pytest --fixtures
# 显示帮助信息
$ pytest -h | --help

# 说明：可以输出用例更加详细的执行信息，比如用例所在的文件及用例名称等
$ pytest -v 
# 说明：输出用例中的调试信息，比如控制台打印信息等
$ pytest -s 
# 说明：标记，执行特定的测试用例
$ pytest -m 
# 说明："关键字"执行用例包含“关键字”的用例
$ pytest -k 
# 说明：简化控制台的输出，两个.. 点代替了 pass 结果
$ pytest -q 
# 控制台输入不展示警告信息
$ pytest -v -p no:warnings --color=yes

# 按 features 运行测试用例
$ pytest --alluredir=report/xml --allure_features=测试登录功能 test_case.py

# 按 story 运行测试用例
$ pytest --alluredir=report/xml --allure_stories=测试登录成功的场景 test_case.py

# 按 severity 运行测试用例
$ pytest --alluredir=report/xml --allure_severities=blocker test_case.py

""" 指定失败次数后停止
"""
# 用例失败1次后停止执行
$ pytest -x
# 用例失败5次后停止执行
$ pytest --maxfail=5

""" 运行指定测试用例
"""
$ pytest test_demo.py
# 指定目录
$ pytest test/
# 指定关键字：包含 demo 且 不包含 foo 的用例
# and not 是表达式
$ pytest -k "demo and not foo"
# 通过 :: 指定
$ pytest test_demo.py::TestClass::test_method
# 通过标签指定 @pytest.mark.smoke
$ pytest -m smoke

""" 修改 traceback 的打印
"""
# 在回溯信息中显示本地变量
$ pytest --showlocals
$ pytest -1 
# 不同信息显示方式，auto 为默认
$ pytest --tb=auto | long | short | line | native | no

""" 生成简略的报告汇总
"""
$ pytest -ra 
# 可追加参数列表如下：
f - failed
E - error
s - skipped
x - xfailed
X - xpassed
p - passed
P - passed with output
a - all except pP

""" 分析测试时间
"""
# 显示最慢的 10 个测试步骤
# 小于 0.01s 不会显示异常，如需显示，追加 -vv 参数
$ pytest --durations=10

# 禁用插件 doctest
$ pytest -p no:doctest

""" 在 python 代码中运行 pytest
"""
pytest.main()
pytest.main(['-x', 'testdir'])
# 指定插件
pytest.main(['-q'], plugins=[MyPlugins()])
```



### 对异常断言

```python 
import pytest

def test_exc():
    with pytest.raises(ZeroDivisionError):
        1 / 0
        
def test_excinfo():
    with pytest.raises(RuntimeError) as excinfo:
        def f():
            f()
    	f()
    assert "Maximun recursion" in str(excinfo.value)

# excinfo 包含了异常的详细信息，属性为 type、value、traceback
# 可以通过 match 参数来正则匹配一个异常发生
def func():
    raise ValueError("Exception 123 raised")
def test_excinfo_match():
    with pytest.raises(ValueError, match=r".* 123 .*"):
        func()
        
        
# pytest.warns 来判断是否产生了特定的警告信息
# conftets.py
def pytest_assertrepr_compare(config, op, left, right)
	# 自定义断言注释
    if isinstance(left, Demo) and isinstance(right, Demo) and op=="==":
        return ["test：", f"{left.val} != {right.val}]
    ...
```





### autouse

- pytest.fixture(autouse=True)表示无需其他声明会自动调用这个 fixture

- autouse fixture 遵守 scope 的定义，如果为 session，则无论定义在哪儿都只会运行一次，如果为 class，则表示在每个 class 中只运行一次。

- 如果在 conftest.py 中定义了 autouse，那么该目录下所有的测试用例都会自动使用该 fixture



谨慎使用该功能，如果只是希望提在项目中提供一个 fixture，而不是想要在每个测试用例中激活使用的，那么可以将 fixture 移动到 conftest.py 中并不要定义 autouse

```python
# conftest.py
@pytest.fixture
def demo(request, db):
	db.begin()
	yield
	db.rollback()

# use
@pytest.mark.usefixtures("demo")
class TestClass:
	def test_method(self)
		...
```

TestClass 内所有的用例都会调用 demo fixture，而其他测试类或测试函数则不会调用，除非他们也添加了 fixture 的引用。



### 重写 fixtures

##### 在 conftest 层重写

```python
tests/
	__init__.py
	conftest.py
        import pytest
        @pytest.fixture
        def username(): 
            return "username"
    test_something.py
		def test_username(username): 
            assert username == "username"
     
	subfolder/
		__init__.py
		conftest.py    
			import pytest
            @pytest.fixture
			def username(username):
				return "overridden‐username"
		test_something.py
			def test_username(username):
				assert username == "overridden‐username"
```



##### 在 module 层重写

```python
tests/
	__init__.py
	conftest.py
        import pytest
        @pytest.fixture
        def username(): 
            return "username"
    test_something.py
    	@pytest.fixture
        def username():
            return "overridden‐username"
		def test_username(username): 
            assert username == "overridden‐username"
```



##### 参数化重写

```python
tests/
	__init__.py
	conftest.py
        import pytest
        @pytest.fixture
        def username(): 
            return "username"
    test_something.py
    	@pytest.mark,parametrize("username", ['username-demo'])
        def test_username(username):
            assert username == "username-demo"
```



### Monkeypatch

有时我们需要修改函数的全局配置或类似网络访问这些不容易测试的代码。monkeypatch 可以用来安全的设置、删除一个属性、字典项或环境变量。



### 临时文件

- 可以通过 tmp_path 来提供一个唯一的临时文件夹供函数调用。
- tmp_path_factory：是一个 session 作用域的 fixture，可以在任意的 fixture 或者用例中创建任何想要的临时文件
- tmpdir：可以为测试用例提供一个唯一的临时文件夹。
- tmpdir_factory



### 捕获警告信息

```python
import warnings
def api_v1():
    warnings.warn(UserWarning("api v1, should use func from v2"))
    return 1

def test_demo():
    assert apu_v1() == 1
```

运行用例后，会在日志信息内展示警告信息。



-W 参数可以用来控制是否显示警告，甚至控制是否将警告转为 Error

```bash
$ pytest -q test_demo.py -W error::UserWarning
```

将警告信息转为报错。



在 pytest.ini 中也可以控制警告。

```ini
[pytest]
# 忽略所有的用户告警，但是会将其他的所有告警转换成 error
filterwarnings = 
	error
	ignore::UserWarning
	
# 禁止捕获告警
addopts = -p no::warnings

# 废弃告警和将要废弃告警
# 忽略能够匹配所给正则表达式的DeprecationWarning
filterwarnings = 
	ignore:.*U.*mode is deprecated:DeprecationWarning
```

当一个告警与多个配置匹配的话，那么放在最后的配置项生效。



### skip 和 xfail

跳过测试用例

```python
# 方式一 装饰器
@pytest.mark.skip(reason="skip test")
def test_skip():
    ...

# 方式二 代码内
def test_skip2():
    if not flag:
        pytest.skip("flag is False, skip!")
```



跳过整个模块

```python
import sys
import pytest

if not sys.platform.startswith("win"):
    pytest.skip("skipping windows-only tests", allow_module_level=True)
```



可以使用 skipif 控制在某些条件下跳过测试。

```python
import sys
@pytest.mark.skipif(sys.version_info < (3, 6), reason="require python 3.6 or higher")
def test_func():
    ...
```

在查找用例时，如果判断 skipif 的条件是 True，该用例会被跳过。

可以再各个模块中共享 skipif 标记。

```python
# test_module.py
import module_name
minversion = pytest.mark.skipif(
	module_name.__versioninfo__ < (1, 1),
    reason="at least module-1.1 required"
)

@minversion
def test_func():
    ...
    
# test_other_module.py
from test_module import minversion

@minversion
def test_module():
    ...
```

在大型项目中一般将这些共享标记放在一个文件里供其他模块调用。



