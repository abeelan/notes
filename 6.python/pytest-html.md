# [pytest-html](https://pytest-html.readthedocs.io/en/latest "官方文档")

> 测试报告生成插件

### 安装

适用版本：Python >=3.6.

```shell
$ pip install pytest-html
```



### 准备测试代码

```python
# test_login.py
class TestLogin:
    def test_success(self):
        print("login success.")

    def test_fail(self):
        print("login fail.")
```



### 使用

```shell
$ pytest --html=report.html test/test_login.py
# >>> generated html file: file:/report.html
```

执行后在当前目录创建测试报告，得到的报告如下：

- assets
    - style.css
- report.html

![效果图](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211209154335326.png)



### 独立报告

默认情况下，CSS 和图像等资源会与 HTML 分开存储，当报告需要共享其他同事时就不太方便了。通过以下方式生成报告：

```shell
$ pytest --html=report.html --self-contained-html
```

这样执行得到的就是一个包含 CSS 样式和外部资源链接的 HTML 文件。



### 自定义报告

该插件还提供了 Hooks 函数，可以对报告数据、样式做出自定义修改。

```python
# 将标题添加到报告之前调用
pytest_html.hooks.pytest_html_report_title(标题) 

# 将摘要部分添加到报告之前调用
pytest_html.hooks.pytest_html_results_summary（前缀，摘要，后缀）

# 构建结果表头后调用
pytest_html.hooks.pytest_html_results_table_header(表头) 

# 构建结果表附加 HTML 后调用
pytest_html.hooks.pytest_html_results_table_html(报告,数据) 

# 在构建结果表行后调用
pytest_html.hooks.pytest_html_results_table_row（报告，单元格）
```

##### 修改标题

测试报告内标题默认为 HTML 文件名。

```python
# conftest.py
def pytest_html_report_title(report):
    report.title = "我是测试报告"
```

![自定义标题](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211209160530222.png)

##### 修改环境（Environment）

测试报告内展示测试运行环境配置相关的信息。

```python
# conftest.py
import pytest

def pytest_configure(config):
    """ 测试运行之前修改环境参数 """
    config._metadata["测试环境"] = "线上"  # 新增
    config._metadata.pop("JAVA_HOME")  # 移除
    
@pytest.hookimpl(tryfirst=True)
def pytest_sessionfinish(session, exitstatus):
    """ 测试运行之后修改环境参数 """
    session.config._metadata["测试状态"] = "已完成"
    session.config._metadata.pop("Base URL")
```

环境表内的 `Base URL` 项，是运行测试之后，环境管理插件自动生成的参数，需要放在运行测试之后的 hook 函数内删除，否则会报错找不到这个 key，而 `JAVA_HOME` 是环境变量，运行测试之前就存在，放在第一个 hook 函数内是可以成功删除的。

![修改环境参数](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211209160013603.png)



##### 修改汇总信息（Summary）

```python
# conftest.py
# Pycharm 可能会出报错提示，不影响执行
from py.xml import html

def pytest_html_results_summary(prefix, summary, postfix):
    prefix.extend([html.p("hello world！")])
```

![插入内容](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211209161328147.png)



##### 修改结果表格（Results）

1. 修改表格：插入列、删除列

```python
# conftest.py
import pytest
from py.xml import html
from datetime import datetime

def pytest_html_results_table_header(cells):
    """ 修改表头属性，增加 Time 列，删除 Link 列 """
    cells.insert(1, html.th("Time", class_="sortable time", col="time"))
    cells.pop()  # 删除最后一列

def pytest_html_results_table_row(report, cells):
    """ 修改表格属性，同步表头修改"""
    cells.insert(1, html.td(datetime.utcnow(), class_="col-time"))
    cells.pop()
```

![效果图](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211209165343334.png)


2. 删除表格中所有通过的结果

```python
# conftest.py
def pytest_html_results_table_row(report, cells):
    """ 修改表格属性
    用例状态（不区分大小写）：
    	passed、skipped、failed、error、xfailed、xpassed、rerun
    """
    if report.passed:
        del cells[:]
```

3. 根据用例执行结果指定日志输出

```python
# conftest.py
def pytest_html_results_table_html(report, data):
    # 通过的用例，指定日志输出内容
    if report.passed:
        del data[:]
        data.append(html.div("Case PASS.", class_="empty log"))
```

##### 向表格内添加内容

```python
# conftest.py
@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    """
    HTML：extra.html('<div>Additional HTML</div>')
    JSON：extra.json({'name': 'pytest'})
    TEXT：extra.text('Add some simple Text')
    LINK：extra.url('http://www.example.com/')
    IMAGE：
        extra.image(image, mime_type='image/gif', extension='gif')
        extra.image('/path/to/file.png')
        extra.image('http://some_image.png')
        extra.png/jpg/svg(image)
    """
    pytest_html = item.config.pluginmanager.getplugin("html")
    outcome = yield
    report = outcome.get_result()
    extra = getattr(report, "extra", [])
    if report.when == "call":
        # always add url to report
        extra.append(pytest_html.extras.url("http://www.example.com/"))
        xfail = hasattr(report, "wasxfail")
        if (report.skipped and xfail) or (report.failed and not xfail):
            # only add additional html on failure
            extra.append(pytest_html.extras.html("<div>Additional HTML</div>"))
        report.extra = extra
```

也可以通过 `fixture` 来实现：

```python
# test_login.py
from pytest_html import extras

def test_success(self, extra):
    print("login success.")
    extra.append(extras.text("login Success"))
```