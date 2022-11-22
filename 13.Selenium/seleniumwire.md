> SeleniumWire 是 Selenium 的扩展，提供访问浏览器发出的底层请求的能力，编写代码方式不变，增加额外的 API 来检查请求和响应并即时更改它们。



官方文档：https://pypi.org/project/selenium-wire/

## 安装

```bash
$ pip install selenium-wire
```



**依赖 OpenSSL**

```bash
# Linux
# For apt based Linux systems
sudo apt install openssl
# For RPM based Linux systems
sudo yum install openssl
# For Linux alpine
sudo apk add openssl

# Mac
$ brew install openssl

# windows 无需安装

# check
$ openssl version
LibreSSL 2.8.3
```



简单使用

```python
from seleniumwire import webdriver

driver = webdriver.Chrome()
driver.get('https://www.example.com')

# Access requests via the `requests` attribute
for request in driver.requests:
    if request.response:
        print(
            request.method,
            request.url,
            request.response.status_code,
            request.response.headers['Content-Type'],
            request.response.body,
            request.response.date
        )
driver.quit()
```



## 特性

- 纯 Python，用户友好的 API
- 捕获 HTTP 和 HTTPS 请求
- 拦截请求和响应
- 即时修改标题、参数、正文内容
- 捕获 websocket 消息
- 支持 HAR 格式
- 代理服务器支持



## 兼容性

- Python 3.7+
- Selenium 4.0.0+
- 支持 Chrome、Firefox、Edge 和 Remote Webdriver



