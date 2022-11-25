> [SeleniumWire](https://github.com/wkeeling/selenium-wire "seleniumwire") 是 Selenium 的扩展，可以获取浏览器发出的请求数据。编写代码方式不变，增加额外的 API 来检查请求和响应并即时更改它们。



## 特性

- 纯 Python，用户友好的 API
- 捕获 `HTTP` 和 `HTTPS` 请求
- 拦截请求和响应
- 即时修改标题、参数、正文内容
- 捕获 `websocket` 消息
- 支持 `HAR` 格式
- 代理服务器支持

兼容性：

- Python 3.7+
- Selenium 4.0.0+
- 支持 Chrome、Firefox、Edge 和 Remote Webdriver



## 安装

```bash
# python 库安装
$ pip install selenium-wire

# 需要 OpenSSL 来解密 HTTPS 请求。
# Linux
# For apt based Linux systems
$ sudo apt install openssl
# For RPM based Linux systems
$ sudo yum install openssl
# For Linux alpine
$ sudo apk add openssl
# For Mac
$ brew install openssl
# windows 无需安装

# check
$ openssl version
LibreSSL 2.8.3
```



## 使用

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



##### 远程浏览器

TODO：目前报错，需要启动一个远程代理服务，接收请求

```python
options = {
    'addr': '127.0.0.1'  
}
driver = webdriver.Remote(
    command_executor='http://www.example.com',
    seleniumwire_options=options
)
```



##### 访问请求

```python
driver.requests  # 按时间顺序排列的捕获请求列表
driver.last_request  # 最近捕获的请求, driver.requests[-1]
```



等待匹配的请求返回

```python
# pat 是一个正则表达式，返回找到的第一个请求
# 未在指定时间返回，引发异常 TimeoutException
driver.wait_for_request(pat, timeout=10)

# 点击页面后，等待返回 12345 接口
button_element.click()
request = driver.wait_for_request('/api/products/12345/')
```



将请求保存为 har 格式

```python
# 必须设置浏览器：enable_har=true
driver.har

# 返回捕获请求的迭代器。在处理大量请求时很有用。
driver.iter_requests()

# 用于设置请求拦截器。请参阅拦截请求和响应。
driver.request_interceptor

# 用于设置响应拦截器。
driver.response_interceptor

# 要清除以前捕获的请求和 HAR 条目，请使用del
del driver.requests
```



Selenium Wire 默认忽略 OPTIONS 请求，因为这些通常是无趣的，只会增加开销。如果要捕获 OPTIONS 请求，则需要将`ignore_http_methods` [选项](https://github.com/wkeeling/selenium-wire#all-options)设置为`[]`.





## 请求对象

请求对象具有以下属性。

- `body`

    请求正文为`bytes`. 如果请求没有正文，则 的值为`body`空，即`b''`.

- `cert`

    字典格式的有关服务器 SSL 证书的信息。对于非 HTTPS 请求为空。

- `date`

    发出请求的日期时间。

- `headers`

    请求标头的类似字典的对象。标头不区分大小写，并且允许重复。请求`request.headers['user-agent']`将返回`User-Agent`标头的值。如果您希望替换标题，请确保先使用 删除现有标题`del request.headers['header-name']`，否则您将创建一个副本。

- `host`

    请求主机，例如`www.example.com`

- `method`

    HTTP 方法，例如`GET`或`POST`等。

- `params`

    请求参数字典。如果同名参数在请求中多次出现，它在字典中的值将是一个列表。

- `path`

    请求路径，例如`/some/path/index.html`

- `querystring`

    查询字符串，例如`foo=bar&spam=eggs`

- `response`

    与请求关联的[响应对象。](https://github.com/wkeeling/selenium-wire#response-objects)`None`如果请求没有响应，就会出现这种情况。

- `url`

    请求网址，例如`https://www.example.com/some/path/index.html?foo=bar&spam=eggs`

- `ws_messages`

    如果请求是 websocket 握手请求（通常以 URL 开头`wss://`），`ws_messages`则将包含发送和接收的所有 websocket 消息的列表。请参阅[WebSocketMessage 对象](https://github.com/wkeeling/selenium-wire#websocketmessage-objects)。

请求对象具有以下方法。

- `abort(error_code=403)`

    使用提供的错误代码触发立即终止请求。在请求拦截器中使用。请参阅[示例：阻止请求](https://github.com/wkeeling/selenium-wire#example-block-a-request)。

- `create_response(status_code, headers=(), body=b'')`

    创建一个响应并返回它而不向远程服务器发送任何数据。在请求拦截器中使用。请参阅[示例：模拟响应](https://github.com/wkeeling/selenium-wire#example-mock-a-response)。



### WebSocketMessage 对象

这些对象表示在浏览器和服务器之间发送的 websocket 消息，反之亦然。它们由`request.ws_messages`websocket 握手请求保存在列表中。它们具有以下属性。

- `content`

    消息内容可以是`str`或`bytes`。

- `date`

    消息的日期时间。

- `from_client`

    `True`消息何时由客户端发送以及`False`何时由服务器发送。



## 响应对象

响应对象具有以下属性。

- `body`

    响应主体为`bytes`. 如果响应没有正文，则 的值为`body`空，即`b''`。有时正文可能已被服务器压缩。`disable_encoding` [您可以使用选项](https://github.com/wkeeling/selenium-wire#all-options)来防止这种情况。要手动解码编码的响应主体，您可以执行以下操作：

```
from seleniumwire.utils import decode

body = decode(response.body, response.headers.get('Content-Encoding', 'identity'))
```

- `date`

    收到响应的日期时间。

- `headers`

    类似字典的响应标头对象。标头不区分大小写，并且允许重复。请求`response.headers['content-length']`将返回`Content-Length`标头的值。如果您希望替换标题，请确保先使用 删除现有标题`del response.headers['header-name']`，否则您将创建一个副本。

- `reason`

    原因短语，例如`OK`或`Not Found`等。

- `status_code`

    响应的状态代码，例如`200`或`404`等。



## 拦截请求和响应

除了捕获请求和响应，Selenium Wire 还允许您使用拦截器动态修改它们。拦截器是一个函数，在请求和响应通过 Selenium Wire 时被调用。在拦截器中，您可以根据需要修改请求和响应。

`driver.request_interceptor`在开始使用驱动程序之前，您可以使用和`driver.response_interceptor`属性设置拦截器函数。请求拦截器应该接受请求的单个参数。响应拦截器应该接受两个参数，一个用于原始请求，一个用于响应。



### 示例：添加请求头

```python
def interceptor(request):
    request.headers['New-Header'] = 'Some Value'

driver.request_interceptor = interceptor
driver.get(...)

# All requests will now contain New-Header
```

如何检查标头是否已正确设置？您可以在页面加载后使用 打印捕获的请求的标头`driver.requests`，或者将网络驱动程序指向https://httpbin.org/headers，这会将请求标头回显给浏览器，以便您可以查看它们。



### 示例：替换现有请求标头

HTTP 请求中允许重复的标头名称，因此在设置替换标头之前，您必须先使用`del`以下示例中的 like 删除现有标头，否则将存在两个具有相同名称的标头（`request.headers`是一个特殊的类字典对象，允许重复).

```python
def interceptor(request):
    del request.headers['Referer']  # Remember to delete the header first
    request.headers['Referer'] = 'some_referer'  # Spoof the referer

driver.request_interceptor = interceptor
driver.get(...)

# All requests will now use 'some_referer' for the referer
```



### 示例：添加响应标头

```python
def interceptor(request, response):  # A response interceptor takes two args
    if request.url == 'https://server.com/some/path':
        response.headers['New-Header'] = 'Some Value'

driver.response_interceptor = interceptor
driver.get(...)

# Responses from https://server.com/some/path will now contain New-Header
```



### 示例：添加请求参数

请求参数与标头的工作方式不同，因为它们是在请求中设置时计算的。这意味着您首先必须读取它们，然后更新它们，然后再将它们写回——就像下面的例子一样。参数保存在常规字典中，因此具有相同名称的参数将被覆盖。

```python
def interceptor(request):
    params = request.params
    params['foo'] = 'bar'
    request.params = params

driver.request_interceptor = interceptor
driver.get(...)

# foo=bar will be added to all requests
```



### 示例：更新 POST 请求正文中的 JSON

```python
import json

def interceptor(request):
    if request.method == 'POST' and request.headers['Content-Type'] == 'application/json':
        # The body is in bytes so convert to a string
        body = request.body.decode('utf-8')
        # Load the JSON
        data = json.loads(body)
        # Add a new property
        data['foo'] = 'bar'
        # Set the JSON back on the request
        request.body = json.dumps(data).encode('utf-8')
        # Update the content length
        del request.headers['Content-Length']
        request.headers['Content-Length'] = str(len(request.body))

driver.request_interceptor = interceptor
driver.get(...)
```



### 示例：基本身份验证

如果站点需要用户名/密码，您可以使用请求拦截器为每个请求添加身份验证凭据。这将阻止浏览器显示用户名/密码弹出窗口。

```python
import base64

auth = (
    base64.encodebytes('my_username:my_password'.encode())
    .decode()
    .strip()
)

def interceptor(request):
    if request.host == 'host_that_needs_auth':
        request.headers['Authorization'] = f'Basic {auth}'

driver.request_interceptor = interceptor
driver.get(...)

# Credentials will be transmitted with every request to "host_that_needs_auth"
```



### 示例：阻止请求

您可以使用它`request.abort()`来阻止请求并将立即响应发送回浏览器。可以提供可选的错误代码。默认值为 403（禁止）。

```python
def interceptor(request):
    # Block PNG, JPEG and GIF images
    if request.path.endswith(('.png', '.jpg', '.gif')):
        request.abort()

driver.request_interceptor = interceptor
driver.get(...)

# Requests for PNG, JPEG and GIF images will result in a 403 Forbidden
```



### 示例：模拟响应

您可以使用`request.create_response()`将自定义回复发送回浏览器。不会向远程服务器发送任何数据。

```python
def interceptor(request):
    if request.url == 'https://server.com/some/path':
        request.create_response(
            status_code=200,
            headers={'Content-Type': 'text/html'},  # Optional headers dictionary
            body='<html>Hello World!</html>'  # Optional body
        )

driver.request_interceptor = interceptor
driver.get(...)

# Requests to https://server.com/some/path will have their responses mocked
```

*您还有其他您认为有用的示例吗？欢迎提交 PR。*



### 取消设置拦截器

要取消设置拦截器，请使用`del`：

```
del driver.request_interceptor
del driver.response_interceptor
```



## 限制请求捕获

Selenium Wire 的工作原理是通过它在后台启动的内部代理服务器重定向浏览器流量。当请求流经代理时，它们会被拦截和捕获。捕获请求可以稍微减慢速度，但您可以采取一些措施来限制捕获的内容。

- `driver.scopes`

    这接受将匹配要捕获的 URL 的正则表达式列表。它应该在发出任何请求之前在驱动程序上设置。当为空（默认）时，将捕获所有 URL。`driver.scopes = [    '.*stackoverflow.*',    '.*github.*' ] driver.get(...)  # Start making requests # Only request URLs containing "stackoverflow" or "github" will now be captured`请注意，即使请求超出范围且未被捕获，它仍将通过 Selenium Wire 传输。

- `seleniumwire_options.disable_capture`

    使用此选项关闭请求捕获。请求仍将通过 Selenium Wire 和您配置的任何上游代理传递，但不会被拦截或存储。请求拦截器不会执行。`options = {    'disable_capture': True  # Don't intercept/store any requests } driver = webdriver.Chrome(seleniumwire_options=options)`

- `seleniumwire_options.exclude_hosts`

    使用此选项可以完全绕过 Selenium Wire。对此处列出的地址发出的任何请求都将直接从浏览器发送到服务器，而不涉及 Selenium Wire。请注意，如果您配置了上游代理，那么这些请求也将绕过该代理。`options = {    'exclude_hosts': ['host1.com', 'host2.com']  # Bypass Selenium Wire for these hosts } driver = webdriver.Chrome(seleniumwire_options=options)`

- `request.abort()`

    [您可以通过在请求拦截器](https://github.com/wkeeling/selenium-wire#intercepting-requests-and-responses)中使用`request.abort()`from 来提前中止请求。这将立即向客户端发送响应，而无需进一步传输请求。您可以使用此机制来阻止某些类型的请求（例如图像）以提高页面加载性能。`def interceptor(request):    # Block PNG, JPEG and GIF images    if request.path.endswith(('.png', '.jpg', '.gif')):        request.abort() driver.request_interceptor = interceptor driver.get(...)  # Start making requests`



## 请求存储

默认情况下，捕获的请求和响应存储在系统临时文件夹（`/tmp`在 Linux 上，通常`C:\Users\<username>\AppData\Local\Temp`在 Windows 上）中名为`.seleniumwire`. 要更改`.seleniumwire`文件夹的创建位置，您可以使用以下`request_storage_base_dir`选项：

```
options = {
    'request_storage_base_dir': '/my/storage/folder'  # .seleniumwire will get created here
}
driver = webdriver.Chrome(seleniumwire_options=options)
```



### 内存存储

Selenium Wire 还支持仅在内存中存储请求和响应，这在某些情况下可能很有用 - 例如，如果您运行的是短期 Docker 容器并且不希望磁盘持久性的开销。`request_storage`您可以通过将选项设置为启用内存存储`memory`：

```
options = {
    'request_storage': 'memory'  # Store requests and responses in memory only
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

如果您担心可能消耗的内存量，您可以限制使用该`request_storage_max_size`选项存储的请求数：

```
options = {
    'request_storage': 'memory',
    'request_storage_max_size': 100  # Store no more than 100 requests in memory
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

当达到最大大小时，旧请求将在新请求到达时被丢弃。`driver.requests`请记住，如果您限制存储的请求数量，当您使用或`driver.wait_for_request()`等检索它们时，请求可能已经从存储中消失。



## 代理

如果您正在访问的站点位于代理服务器后面，您可以在传递给网络驱动程序的选项中告诉 Selenium Wire 有关该代理服务器的信息。

配置采用以下格式：

```
options = {
    'proxy': {
        'http': 'http://192.168.10.100:8888',
        'https': 'https://192.168.10.100:8888',
        'no_proxy': 'localhost,127.0.0.1'
    }
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

要将 HTTP Basic Auth 与您的代理一起使用，请在 URL 中指定用户名和密码：

```
options = {
    'proxy': {
        'https': 'https://user:pass@192.168.10.100:8888',
    }
}
```

对于 Basic 以外的身份验证，您可以`Proxy-Authorization`使用该`custom_authorization`选项为标头提供完整值。例如，如果您的代理使用 Bearer 方案：

```
options = {
    'proxy': {
        'https': 'https://192.168.10.100:8888',  # No username or password used
        'custom_authorization': 'Bearer mytoken123'  # Custom Proxy-Authorization header value
    }
}
```

`Proxy-Authorization`可以在[此处](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authorization)找到有关标头的更多信息。

代理配置也可以通过名为`HTTP_PROXY`,`HTTPS_PROXY`和的环境变量加载`NO_PROXY`：

```
$ export HTTP_PROXY="http://192.168.10.100:8888"
$ export HTTPS_PROXY="https://192.168.10.100:8888"
$ export NO_PROXY="localhost,127.0.0.1"
```



### Socket

使用 SOCKS 代理与使用基于 HTTP 的代理相同，但您将方案设置为`socks5`：

```
options = {
    'proxy': {
        'http': 'socks5://user:pass@192.168.10.100:8888',
        'https': 'socks5://user:pass@192.168.10.100:8888',
        'no_proxy': 'localhost,127.0.0.1'
    }
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

如果您的代理不需要身份验证，您可以省略`user`和。`pass`

以及`socks5`，计划`socks4`和`socks5h`支持。`socks5h`当您希望 DNS 解析发生在代理服务器而不是客户端时使用。

**将 Selenium Wire 与 Tor 结合使用**

如果您想使用 Tor 运行 Selenium Wire，请参阅[此示例。](https://gist.github.com/woswos/38b921f0b82de009c12c6494db3f50c5)



### 动态切换

如果要更改现有驱动程序实例的代理设置，请使用`driver.proxy`属性：

```
driver.get(...)  # Using some initial proxy

# Change the proxy
driver.proxy = {
    'https': 'https://user:pass@192.168.10.100:8888',
}

driver.get(...)  # These requests will use the new proxy
```

要清除代理，请设置`driver.proxy`为空 dict `{}`。

该机制还支持`no_proxy`和`custom_authorization`选项。



## 机器人检测

如果 Selenium Wire 在您的环境中找到它，它将与[undetected-chromedriver集成。](https://github.com/ultrafunkamsterdam/undetected-chromedriver)该库将透明地修改 ChromeDriver 以防止其在网站上触发反机器人措施。

如果你想利用这一点，请确保你已经安装了 undetected_chromedriver：

```
pip install undetected-chromedriver
```

然后在您的代码中，导入`seleniumwire.undetected_chromedriver`包：

```
import seleniumwire.undetected_chromedriver as uc

chrome_options = uc.ChromeOptions()

driver = uc.Chrome(
    options=chrome_options,
    seleniumwire_options={}
)
```



## 证书

Selenium Wire 使用它自己的根证书来解密 HTTPS 流量。浏览器通常不需要信任此证书，因为 Selenium Wire 告诉浏览器将其添加为例外。这将允许浏览器正常运行，但它会在地址栏中显示“不安全”消息（和/或解锁的挂锁）。如果您希望摆脱此消息，您可以手动安装根证书。

[您可以在此处](https://github.com/wkeeling/selenium-wire/raw/master/seleniumwire/ca.crt)下载根证书。下载后，导航到浏览器设置中的“证书”，然后在“权限”部分导入证书。



### 使用您自己的证书

如果您想使用自己的根证书，您可以使用`ca_cert`和`ca_key`选项提供证书路径和私钥。

如果您确实指定了自己的证书，请务必手动删除 Selenium Wire 的[临时存储文件夹](https://github.com/wkeeling/selenium-wire#request-storage)。这将清除可能从以前的运行中缓存的任何现有证书。



## 所有选项

可以通过`seleniumwire_options`webdriver 属性传递给 Selenium Wire 的所有选项的摘要。

- `addr`

    运行 Selenium Wire 的机器的 IP 地址或主机名。这默认为 127.0.0.1。如果您使用的是[远程 webdriver](https://github.com/wkeeling/selenium-wire#creating-the-webdriver) ，您可能希望将其更改为机器（或容器）的公共 IP 。

```
options = {
    'addr': '192.168.0.10'  # Use the public IP of the machine
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `auto_config`

    Selenium Wire 是否应该为请求捕获自动配置浏览器。`True`默认。

- `ca_cert`

    如果您更喜欢使用自己的证书而不是使用默认证书，则为根 (CA) 证书的路径。

```
options = {
    'ca_cert': '/path/to/ca.crt'  # Use own root certificate
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `ca_key`

    如果您使用自己的根证书，则为私钥的路径。使用您自己的证书时，必须始终提供密钥。

```
options = {
    'ca_key': '/path/to/ca.key'  # Path to private key
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `disable_capture`

    禁用请求捕获。当`True`没有任何内容被拦截或存储时。`False`默认。

```
options = {
    'disable_capture': True  # Don't intercept/store any requests.
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `disable_encoding`

    要求服务器发回未压缩的数据。`False`默认。当`True`这将`Accept-Encoding`标头设置`identity`为所有出站请求时。请注意，它并不总是有效 - 有时服务器可能会忽略它。

```
options = {
    'disable_encoding': True  # Ask the server not to compress the response
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `enable_har`

    当`True`保留 HTTP 事务的 HAR 存档时，可以使用`driver.har`. `False`默认。

```
options = {
    'enable_har': True  # Capture HAR data, retrieve with driver.har
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `exclude_hosts`

    应完全绕过 Selenium Wire 的地址列表。请注意，如果您配置了上游代理，那么对排除的主机的请求也将绕过该代理。

```
options = {
    'exclude_hosts': ['google-analytics.com']  # Bypass these hosts
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `ignore_http_methods`

    Selenium Wire 应忽略且不捕获的 HTTP 方法列表（指定为大写字符串）。默认是`['OPTIONS']`忽略所有 OPTIONS 请求。要捕获所有请求方法，请设置`ignore_http_methods`为空列表：

```
options = {
    'ignore_http_methods': []  # Capture all requests, including OPTIONS requests
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `port`

    Selenium Wire 后端监听的端口号。您通常不需要指定端口，因为系统会自动选择随机端口号。

```
options = {
    'port': 9999  # Tell the backend to listen on port 9999 (not normally necessary to set this)
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `proxy`

    如果您使用代理，则为上游[代理服务器配置。](https://github.com/wkeeling/selenium-wire#proxies)

```
options = {
    'proxy': {
        'http': 'http://user:pass@192.168.10.100:8888',
        'https': 'https://user:pass@192.168.10.100:8889',
        'no_proxy': 'localhost,127.0.0.1'
    }
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `request_storage`

    要使用的存储类型。Selenium Wire 默认为基于磁盘的存储，但您可以通过将此选项设置为以下来切换到内存存储`memory`：

```
options = {
    'request_storage': 'memory'  # Store requests and responses in memory only
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `request_storage_base_dir`

    Selenium Wire 在使用其默认的基于磁盘的存储时存储捕获的请求和响应的基本位置。这默认为系统临时文件夹（`/tmp`在 Linux 上，通常`C:\Users\<username>\AppData\Local\Temp`在 Windows 上）。将在此处创建一个名为的子文件夹`.seleniumwire`来存储捕获的数据。

```
options = {
    'request_storage_base_dir': '/my/storage/folder'  # .seleniumwire will get created here
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `request_storage_max_size`

    使用内存存储时要存储的最大请求数。默认无限制。使用默认的基于磁盘的存储时，此选项当前无效。

```
options = {
    'request_storage': 'memory',
    'request_storage_max_size': 100  # Store no more than 100 requests in memory
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `suppress_connection_errors`

    是否抑制与连接相关的回溯。`True`默认情况下，这意味着有时在浏览器关闭时发生的无害错误不会提醒用户。当被抑制时，连接错误消息被记录在 DEBUG 级别，没有回溯。设置为`False`允许异常传播并查看完整的回溯。

```
options = {
    'suppress_connection_errors': False  # Show full tracebacks for any connection errors
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

- `verify_ssl`

    是否应验证 SSL 证书。`False`默认情况下，这可以防止自签名证书出错。

```
options = {
    'verify_ssl': True  # Verify SSL certificates but beware of errors with self-signed certificates
}
driver = webdriver.Chrome(seleniumwire_options=options)
```

