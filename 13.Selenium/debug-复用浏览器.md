> selenium debug 模式，远程调试，复用浏览器。



### 场景

比如测试京东购物流程，必须扫码登录成功后，才能进行后续操作。

用例编写调试时，每次运行都会打开一个新的浏览器窗口，得重新扫码登录才行。

原因是`ChromeDriver`默认每次被调用启动时都会加载一个新的会话，像这种频繁调试的场景，就比较浪费时间。

对此，`ChromeDriver`通过开放远程端口提供`debug`的能力。

将 [ChromeOptions](https://sites.google.com/a/chromium.org/chromedriver/capabilities "ChromeOptions") 对象中的`debuggerAddress`参数，设置为要连接的调试器服务器地址，格式为：

`<hostname/ip:port>`

即可，后续操作都在当前窗口进行，达到浏览器复用的目的。



### 配置


1. 将`chromedriver`添加到环境变量；

```Python
# bin 目录已经配置好环境变量
# 这里可以直接将下载好的 driver 放到该目录下
$ mv chromedriver /usr/local/bin
```

2. 设置 - 隐私设置和安全性 - 网站设置 - 后台同步 - 默认行为 - 勾选 最近关闭的网站可以完成数据收发操作；

3. 完全关闭浏览器，点击鼠标右键 - 退出；

4. 开启浏览器远程调试端口，这个端口可以自定义，不与本地已开端口冲突就行；

```Python
$ /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome -remote-debugging-port=9222
# 正在现有的浏览器会话中打开。
# 如果出现上面提示，重新关闭浏览器，执行命令
```

5. 执行脚本，在`options`内添加远程浏览器的`IP`地址和端口号，执行测试。

```python
@pytest.fixture(scope="session")
def driver():
    options = Options()
    # 设置复用浏览器的端口，地址
    options.debugger_address = "127.0.0.1:9222"
    driver = webdriver.Chrome(options=options)
    driver.get("https://baidu.com")
    yield driver
    driver.quit()


def test_demo(driver):
    driver.find_element(By.ID, "kw").send_keys("selenium")
```



这时不管运行多少次测试用例，都复用同一个调试浏览器，不会打开新的浏览器窗口。