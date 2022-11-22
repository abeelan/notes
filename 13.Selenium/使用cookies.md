> WebDriver 与 cookie 进行交互的方法。



### cookie

通常用于网站识别用户的身份，保持登录状态或追踪用户浏览记录。

- Name：cookie 的名称
- Value：cookie 的值
- Domain：允许接收 cookie 的主机
- Path：请求 URL 路径
- Expires/Max-Age：cookie 有效期
- Size：cookie 的大小
- HttpOnly：是否仅能通过 HTTP 请求（true/false）
- Secure：是否仅通过 HTTPS 请求（true/false）
- SameSite：是否限制第三方 cookie
- Priority：优先级，包含低、中（默认）或高值



### 添加 cookie

添加操作只接收一组定义的、可序列化的`JSON`对象。

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("http://www.example.com")

# Adds the cookie into current browser context
driver.add_cookie({"name": "foo", "value": "bar"})
```



### 获取 cookie

根据`name`获取单条`cookie`。

```python
driver.add_cookie({"name": "foo", "value": "bar"})
print(driver.get_cookie("foo"))

"""
{'domain': 'example.com', 'httpOnly': False, 'name': 'foo', 'path': '/', 'secure': False, 'value': 'bar'}
"""
```



获取所有 cookie。

```python
driver.add_cookie({"name": "test1", "value": "cookie1"})
driver.add_cookie({"name": "test2", "value": "cookie2"})
print(driver.get_cookies())

"""
[
    {'domain': 'example.com', 'httpOnly': False, 'name': 'test2', 'path': '/', 'secure': False, 'value': 'cookie2'}, 
    {'domain': 'example.com', 'httpOnly': False, 'name': 'test1', 'path': '/', 'secure': False, 'value': 'cookie1'}
]
"""
```



### 删除 cookie

根据 name 删除单条 cookie。

```python
driver.add_cookie({"name": "test1", "value": "cookie1"})
driver.add_cookie({"name": "test2", "value": "cookie2"})
driver.delete_cookie("test1")
print(driver.get_cookies())

"""
[
	{'domain': 'example.com', 'httpOnly': False, 'name': 'test2', 'path': '/', 'secure': False, 'value': 'cookie2'}
]
"""
```



删除所有 cookie。

```python 
driver.add_cookie({"name": "test1", "value": "cookie1"})
driver.add_cookie({"name": "test2", "value": "cookie2"})
driver.delete_all_cookies()
print(driver.get_cookies())

"""
[]
"""
```



### cookie 保存与读取

在项目中，通常把获取到的 `cookie` 保存到文件，在调用处直接读取即可。

```python
"""
cookies 保存到文件
"""
driver.get("已经登录后的网站")
cookies = driver.get_cookies()
with open("cookies.yaml", "w", encoding="utf-8") as f:
	yaml.dump(cookies, f)

"""
使用 cookies 时从文件进行读取
"""
driver.get("重新打开一个未登录的窗口")
with open("cookies.yaml", encoding="utf-8") as f:
    cookies = yaml.safe_load(f)
for cookie in cookies:
    driver.add_cookie(cookie)
```



### SameSite

**SameSite** 是用来限制第三方`Cookie`的属性，该设置防止`CSRF`(跨站请求伪造)攻击和用户追踪。



使用限制：

- chrome(80+version)
- Firefox(79+version)
- Selenium v4+

属性设置：
- Strict：严格模式，完全禁止携带 `cookies` 与第三方网站请求一起发送
- Lax：宽松模式，允许携带 `cookies` 与第三方网站 `GET` 请求一起发送



```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("http://www.example.com")
# 设置方式
driver.add_cookie({"name": "foo1", "value": "value", 'sameSite': 'Strict'})
driver.add_cookie({"name": "foo2", "value": "value", 'sameSite': 'Lax'})
cookie1 = driver.get_cookie('foo1')
cookie2 = driver.get_cookie('foo2')
print(cookie1)
print(cookie2)  
```

