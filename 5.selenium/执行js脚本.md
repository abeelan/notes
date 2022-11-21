> 在当前窗口或框架上下文中，执行 JavaScript 脚本。



使用`JavaScript`操作页面：
- 解决点击不生效的问题
- 页面滚动
- 修改元素属性



### JavaScript

```JavaScript
// 进入浏览器 -> 检查 -> console
// 获取网页名称
document.title

// 显示 alert
window.alert("hello selenium")

// 获取网页的性能数据
JSON.stringify(performance.timing)
```



### Selenium 调用

```python
def execute_script(self, script, *args):
    # script: JavaScript 代码
   	# args: 任何适用于 JavaScript 的参数
    ...
    
# 示例：返回 h1 标签元素的文本
driver.get("https://www.selenium.dev/")
header = driver.find_element(By.CSS_SELECTOR, "h1")
# return: 返回 js 执行结果
# arguments: 参数传递
text = driver.execute_script('return arguments[0].innerText', header)
assert text == "Selenium automates browsers. That's it!"
```



#####  定位元素

```Python
js = 'return document.getElementById("su")'
driver.execute_script(js)
```



##### 滑动

常见的滑动场景分为四种：

- 滑动至底部
- 滑动至顶部
- 滑动至具体位置
- 滑动至目标元素可见

```Python
# 模拟鼠标滚轮，滑动页面至底部
js = "window.scrollTo(0, document.body.scrollHeight)" 
driver.execute_script(js)

# 模拟鼠标滚轮，滑动页面至顶部
js = "window.scrollTo(0, 0)"
driver.execute_script(js)
js = "window.scrollBy(0, 500)"  # 向下滑动500个像素
js = "window.scrollBy(0, -500)"　# 向上滚动500个像素
js = "window.scrollBy(500, 0)"  # 向右滑动500个像素
js = "window.scrollBy(-500, 0)"　# 向左滚动500个像素

# 滑动到具体位置
driver.execute_script("window.scrollTo(x, y)")  

# 向下滚动至-元素可见
driver.execute_script("arguments[0].scrollIntoView();", element)
 
# 向上滚动至-元素可见
driver.execute_script("arguments[0].scrollIntoView(false);", element)
```



##### 示例：操作控件 & 获取返回值

```Python
# 场景：百度搜索结果页，滑动到页面底部，点击下一页

"""
1. 进入搜索结果页
"""
driver.get("http://www.baidu.com")
driver.find_element_by_id("kw").send_keys("selenium")
ele_search = driver.execute_script('return document.getElementById("su")')
ele_search.click()

"""
2. 通过 JavaScript 滑动到页面底部
"""
js_code = "document.documentElement.scrollTop=10000"
driver.execute_script(js_code)
sleep(2)
driver.find_element_by_css_selector("#page a:nth-last-child(1)").click()

"""
3. 断言页面跳转，打印页面标题和页面性能数据
"""
# 方法一: 多条 js 脚本分别执行
js_codes = [
    "return document.title",
    "return JSON.stringify(performance.timing)"
]
for code in js_codes:
    print(self.driver.execute_script(code))

# 方法二 合并执行
# 注意，在 title 处已经返回，后续不会执行
js_code = "return document.title;return JSON.stringify(performance.timing)"
title = self.driver.execute_script(js_code)
assert title == "selenium_百度搜索"

# 会打印 timing ，因为 title 未返回
js_code = "document.title;return JSON.stringify(performance.timing)"
print(self.driver.execute_script(js_code))
```



##### 示例：修改控件属性

```Python
"""
时间控件属性为 readonly
手动测试时：手动去选择对应的时间
自动化测试时：使用 js 修改控件属性
	- 要取消日志的 readonly 属性
	- 给 value 赋值

场景：12306 网站内修改出发日期
"""

"""
1. 打开 12306
"""
driver.get("https://www.12306.cn/index/")

"""
2. 修改出发日期为 2021-5-12
"""
driver.execute_script(
    'train_date=document.getElementById("train_date");'
    'train_date.removeAttribute("readonly");'
    'train_date.value = "2021-05-12"'
)

"""
3. 打印出发日期
"""
print(driver.execute_script(
    'return document.getElementById("train_date").value')
)
driver.quit()
```