> requests-html API 使用示例。
>
> [官方文档](https://docs.python-requests.org/projects/requests-html/en/latest/ "Docs")
>
> [Github](https://github.com/psf/requests-html "Github")



移动端混合应用，除 API 接口外，还包含大量的 web 页面需要校验，进行自动化测试。

在 GitHub 上找到一个解析 HTML 页面的库，与 requests 库是同作者，大神出品，必属精品。哈哈，研究一下。


### 安装

仅支持 Python 3.6 及以上版本。

```bash
$ pip install requests-html
```

### 使用

请求数据的方法：

- 发送 web 请求，获取 HTML 响应
- 发送异步请求，异步获取 HTML 响应
- 解析本地 HTML 文件

解析数据的方法：

- links/absolute_links：获取网页链接
- find：基于 CSS 选择器提取
- xpath：基于 xpath 语法提取
- search/search_all：正则获取文本内容

JavaScript 渲染：

- render()
- arender()：异步渲染



### 1、请求数据

请求方法与 requests 一样，因为他们是继承的关系。

```python
# 发送请求
from requests_html import HTMLSession

session = HTMLSession()
r = session.get('https://python.org/')
html = r.html
```

```python
# 异步请求
from requests_html import AsyncHTMLSession

asession = AsyncHTMLSession()
r = await asession.get('https://python.org/')
html = r.html
```

```python
# 本地 HTML 文件
from requests_html import HTML

doc = """<a href='https://httpbin.org'>"""
html = HTML(html=doc)
```

以上三种方式都会得到一个 HTML 对象，后面的数据操作都是基于这个 HTML 对象进行的。



### 2、提取数据

简单了解下 API 的使用方法。

##### Links

获取页面上所有的链接（href 标签）。

```python
from requests_html import HTMLSession

session = HTMLSession()
r = session.get("https://www.baidu.com/")
html = r.html

""" 
相对链接：<a href="/index.html">Home</a>
绝对链接：<a href="https://www.navegabem.com/index.html">Home</a>
"""

# 获取全部链接，包含相对链接
html.links

# 获取绝对链接，自动拼接前缀
html.absolute_links  
```

##### Text

获取页面上所有的文本内容。

```python
session = HTMLSession()
r = session.get("https://www.baidu.com/")
html = r.html

# 元素内和 HTML 标签内的文本内容
html.text

# 包含链接文本内容
html.full_text
```

获取整个页面的文本内容，数据量往往非常大，且不好清洗，所以一般都是对单个元素使用。当对某一个元素使用时，就获取该元素内的文本内容，后面会用到。

##### XPATH

通过 [XPATH 语法](https://www.w3schools.com/xml/xpath_syntax.asp "xpath") 获取数据。

```python
# 获取页面中的书名和作者
# ==> 后面跟的是输出
session = HTMLSession()
r = session.get("https://book.easou.com/ta/index.m")
html = r.html

# 获取第一条匹配的数据，上文中 text 的好处在这里就体现出来了
element = html.xpath("//div[@class='text']", first=True)
element.text

# 获取所有匹配的数据，elements 是一个组
elements = html.xpath("//div[@class='text']")
# 找到一本下标为 5 的书籍，获取该元素内的全部文本信息
elements[5].text.split("\n")
==> ['遮天', '辰东', '古典仙侠', '冰冷与黑暗并存的宇宙深处，九具...', '630.6万字', '古典仙侠']
```



##### CSS

通过 [CSS 选择器](https://www.w3schools.com/cssref/css_selectors.asp "css") 获取数据。

```python
# 获取页面上的 tab 名称
session = HTMLSession()
r = session.get("https://book.easou.com/ta/index.m")
html = r.html

# 获取第一个 tab 的名称
element = html.find("div.banner li", first=True)
element.text
==> '精选'

# 获取全部 tab 名，这是 css selector 的另一种写法
elements = html.find("div[class='banner'] li")
tabs = [ele.text for ele in elements]
==> ['精选', '分类', '免费', '书架']

# 获取包含关键字的 tab 名
elements = html.find("div[class='banner'] li", containing="书")
elements[0].text
==> '书架'
```



##### re

基于正则获取数据。

```python
# 获取排行榜第一页的书名
session = HTMLSession()
r = session.get("https://book.easou.com/ta/rank.m?pageName=排行")
html = r.html

"""
1. search() 返回第一条匹配的数据
"""
# 使用大括号匹配文本，类似 (.*?)
element1 = html.search('<div class="title">{}</div>')
element2 = html.search('<div class="desc">{desc}</div>')

# 返回的是一个对象，包含一个元组，一个字典
# 大括号内为空，未对数据命名，数据放在元组内，通过下标取值
# 大括号内对数据命名，数据放在字典内，仅能通过 dict[key] 取值，不支持 get()
element1
==> <Result ('深空彼岸',) {}>
element2
==> <Result () {'desc': '浩瀚的宇宙中，一片星系的生灭，也不过是...'}>

element1[0]
==> '深空彼岸'
element2["desc"]
==> '浩瀚的宇宙中，一片星系的生灭，也不过是...'

"""
2. search_all() 返回全部数据
"""
elements = html.search_all('<div class="title">{}</div>')
books = [ele[0] for ele in elements]
len(books)
==> 20
```

### 3、JavaScript 渲染

简单说下 JavaScript 渲染。

在浏览器访问一个链接，中间过程省略，浏览器会得到服务器返回的响应内容，开始生成 HTML DOM 树、渲染树，当遇到 JavaScript 脚本，也就是 `<script>` 标签时，会阻塞，等待 js 脚本执行，完成最终页面渲染。

使用代码直接发送页面请求时，无法获取利用 JavaScript 生成的页面，必须借助浏览器内核实现。这里实现渲染能力借助的是 Chromium 无头浏览器，页面渲染时是无感的。

> 引用作者描述
>
> Reloads the response in Chromium, and replaces HTML content with an updated version, with JavaScript executed.



通过百度主页的热搜模块，演示该功能。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220114191324577.png)

```python
# 百度热搜 模块就是通过 js 渲染的
session = HTMLSession()
r = session.get(
    url="https://www.baidu.com/",
    headers={"user-agent": "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Mobile Safari/537.36"}
)
html = r.html

# 查看页面数据
html.html
==> 输出的文本内容里没有包含「百度热搜」关键词

# 进行页面渲染后，再次打印
html.render()
html.html
==> 输出文本中包含「百度热搜」关键词

# 拿到热搜数据
hot_items = html.find("span[class='title-content-title']")
hot_news = [e.text for e in hot_items]
hot_news
==> [
 '讲好冬奥故事 共赴冰雪之约',
 '天津出现45起家庭聚集疫情',
 '春节返乡31省份防疫要求汇总来了',
  ...
]
```

仅第一次运行 render() 方法时，会下载 Chromium。可能会下载失败：

- 受网络影响（下载需要访问外网服务器，不稳定）;
- urllib3 版本号需要小于 1.25，我报错后通过下面命令回退版本后解决。

```bash
# pyppeteer.errors.NetworkError: Execution context was destroyed, most likely because of a navigation.
$ pip install -U "urllib3<1.25"
```



### 4、异步请求

```python
from requests_html import AsyncHTMLSession, HTML

urls = [
    "https://www.baidu.com/",
    "https://python.org/",
    "https://www.bilibili.com/"
]

asession = AsyncHTMLSession()

async def async_html(url) -> HTML:
	r = await asession.get(url, headers=headers)
    html = r.html
    
    # 异步渲染页面
    await html.arender()
	return html

task = [lambda url=url: async_html(url) for url in urls]
html_list: [HTML, ...] = asession.run(*task)
==> [
	<HTML url='https://www.python.org/'>,
	<HTML url='https://www.baidu.com/'>,
	<HTML url='https://www.bilibili.com/'>
]
```

### 小技巧

##### 1、选择器语法调试

通过 XPATH 或 CSS 选择器获取数据时，放在代码里面验证表达式的正确性，刚开始不熟悉，要多次修改表达式，效率很低。

这种情况下可以使用 Chrome 开发者工具的搜索功能，快速验证表达式结果。

使用方法：选中 `Elements`tab 项，调出搜索框（`Ctrl + F`），支持 XPATH 和 CSS 选择器搜索，写入表达式查看命中结果即可。

![如图](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220114173442122.png)

##### 2、如何查看当前页面是否是由 JavaScript 渲染而成的？

设置 Chrome 开启/禁用 JavaScript，直达链接如下：

```
chrome://settings/content/javascript
```

禁用后，访问页面，通过 JavaScript 渲染的页面或者模块是不会展示的。