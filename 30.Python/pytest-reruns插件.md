> [pytest-rerunfailures](https://pypi.org/project/pytest-rerunfailures/#toc-entry-1 "pypi") 插件，功能是用例执行失败可以重试。
>
> - python 3.6+
> - pytest 5.3+



用例重试时，`fixture` 或 `setup_class` 也将被重新执行。



**安装**

```bash
$ pip install pytest-rerunfailures
```



**使用**

命令行使用

```bash
# 用例失败重试 3 次
$ pytest --reruns 3

# 用例失败重试 3 次，每次重试间隔时间为 2 秒
$ pytest --reruns=3 \
  --reruns-delay=2

# 仅重试指定异常的用例
$ pytest --reruns=2 \
  --only-rerun=AssertionError
# 指定多个异常
pytest --reruns 2 \
  --only-rerun AssertionError \
  --only-rerun ValueError

# 仅重试与指定异常不匹配的用例，仅重试非断言失败的用例
$ pytest --reruns 2 \
  --rerun-except AssertionError
# 指定多个
$ pytest --reruns 2 \
  --rerun-except AssertionError \
  --rerun-except OSError
```



用例内使用装饰器

```python
import pytest

"""执定重试次数"""
@pytest.mark.flaky(reruns=2)
def test_example():
    assert False

"""指定重试间隔时间"""
@pytest.mark.flaky(reruns=2, reruns_delay=2)
def test_example():
    assert False
    
"""指定重试条件"""
# linux 系统下开启重试， mac 执行不重试
@pytest.mark.flaky(reruns=2, condition=sys.platform.startswith("Linux"))
def test_example():
    assert False
# 非 linux 系统开启重试，mac 执行重试生效
@pytest.mark.flaky(reruns=2, condition=not sys.platform.startswith("Linux"))
def test_example():
    assert False
```



**兼容性**

- 不能与 `fixture` 一起使用
- 与 `pytest-xdist –looponfail` 自动监听功能不兼容
- 与 –pdb 调试功能不兼容