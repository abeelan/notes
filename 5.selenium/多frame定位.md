### frame 简介

`frame` 是 `html` 中的框架导航。同一个框架集中，点击某一框架的超链接，内容会在另一个框架的窗口中展示。

比如后台管理页面，点击左侧导航栏按钮，在右侧区域展示加载的内容，而不是打开一个新的窗口。

![](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221116175859998.png)

代码如下，可访问在线 html 运行网站编辑查看。
```html
<!-- https://www.w3school.com.cn/tiy/t.asp?f=html_frame_cols -->
<html>
    
<frameset rows="15%,*">
    
    <frame src="/example/html/frame_a.html">
    
    <frameset cols="20%,*">
    <frame src="/example/html/frame_b.html">
    <frame src="/example/html/frame_c.html" name="show">
    </frameset>

</frameset>

</html>
```

在左侧导航栏页面添加 `href` 标签和 `traget` 属性，即可实现页面框架跳转。

```html
<!-- 1、定义框架集中每个框架的名称 -->
<frame src="/example/html/frame_c.html" name="show">
<!-- 2、左侧导航栏页面内添加 target 属性 -->
<a href=url target="show">
```


### fram 定位

`frame`标签包含三种：

- `frameset`和普通的标签定位方式一致，通过`id、name`等常规方式；
- `frame`与`iframe`则需要切换进入`frame`内，才能进行定位。

在`web`自动化中，如果某个元素定位不到，则很有可能是隐藏在`frame`中。


### 多 frame 切换

切换 `frame` 方式如下。

```python
# 通过 id 切换
driver.switch_to.frame('buttonframe')

# 通过 web element 切换
iframe = driver.find_element(By.CSS_SELECTOR, "#modal > iframe")
driver.switch_to.frame(iframe)  

# 多元素通过 index 切换
iframe = driver.find_elements(By.TAG_NAME,'iframe')[1]
driver.switch_to.frame(iframe)

# 回到默认 frame
driver.switch_to.default_content()

# 切换到父 frame
driver.switch_to.parent_frame() 
```



对于嵌套的 `frame`，需要先进入到父节点，再切换进入子节点，然后对子节点进行操作。

```python
driver.switch_to.frame("父节点")
driver.switch_to.frame("子节点")
```



**示例**

```Python
with webdriver.Chrome() as driver:
    driver.get("https://www.runoob.com/try/try.php?filename=jqueryui-api-droppable")

    btn_locator = (By.ID, "draggable")
    # 该元素是「请拖拽我」，运行报错：NoSuchElementException
    # 因为该元素在 iframe 标签内，直接无法定位到元素
    # driver.find_element(*btn_locator).text)

    driver.switch_to.frame("iframeResult")  # 切换到 iframe 内
    assert driver.find_element(*btn_locator).text == "请拖拽我！"

    submit_locator = (By.ID, "submitBTN")
    # 点击提交按钮，运行报错：NoSuchElementException
    # 因为提交按钮在当前 iframe 外，还需要切出去
    # driver.find_element(*submit_locator).click()

    driver.switch_to.parent_frame()  # 切回父 frame
    assert driver.find_element(*submit_locator).text == "点击运行 》"

```