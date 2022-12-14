> Appium 常用 API。



## 元素定位

### DOM 结构

- dom：`Document Object Model`，文档对象模型。
- dom 应用：用于表示界面的控件层级，界面的结构化描述。常见的格式为 `html\xml`。核心元素为节点和属性。



Andoid 应用的层级结构和 `html` 不一样，是一个定制的 `xml`。`App source` 类似于 dom，表示界面所有控件的结构。每个控件都有它的属性，但没有 css 属性。



IOS 与 Android 的 dom 结构类似，但是属性有所区别，例：

- Android resourceid = iOS name
- Android content-des = IOS accessibility-id



### Android



##### Uiautomator 定位表达式

uiautomator 是安卓的工作引擎，比 xpath 速度快。缺点是表达式书写复杂，容易写错，IDE 没有提示。

写法参考：

- [Appium Selector](http://appium.io/docs/en/writing-running-appium/android/uiautomator-uiselector/, "appium selector")

- [Android Selector](https://developer.android.com/reference/androidx/test/uiautomator/UiSelector "android selector")

```python
# 点击按钮（通过文本定位）
driver.find_element_by_android_uiautomator(
    'new UiSelector().text("登录")'
).click()

# 输入文本（通过 ID 定位）
driver.find_element_by_android_uiautomator(
    'new UiSelector().resourceId("login_account")'
).send_keys("123456")

# 点击按钮（通过 ID 定位）
driver.find_element_by_android_uiautomator(
    'new UiSelector().resourceId("button_next")'
).click()

# 点击推荐（组合定位，通过 ID 查找父元素定位子元素）
recommend = 'new UiSelector().resourceId("title_text").childSelector(text("推荐"))'
driver.find_element_by_android_uiautomator(recommend).click()

# 页面滑动找到「猜你喜欢」并点击（滚动查找，直到元素出现）
text = 'new UiScrollable(' \
       'new UiSelector().scrollable(true).instance(0)).scrollIntoView(' \
       'new UiSelector().text("猜你喜欢").instance(0));'
driver.find_element_by_android_uiautomator(text).click()
```



##### Toast 控件定位

简易消息提示框，在当前页面显示一个浮动的显示块，与 `dialog` 不同，它永远不会获得焦点，尽可能不引人注意，同时向用户提供必要的信息，时间到后自动消失。



它是系统级别的控件，`APP` 发送内容和指定时间给系统，由系统弹框，这类的控件不在 `APP` 内，需要特殊的控件识别方法。



**定位原理**

- `appium` 使用 `uiautomator` 底层的机制来分析抓取 `toast`，并且把 `toast` 放到控件树里面，但是它本身并不属于控件
- 获取页面内容 `getPageSource` 无法找到
- 必须使用 `xpath` 查找

```python
driver.find_element_by_xpath("//*[@class='android.widget.Toast']").text)
driver.find_element_by_xpath("//*[contains(@text, '暂无更新')]").text)
```



### iOS



## 元素操作

##### 元素的常用操作

```python
# 点击
ele.click()

# 输入文本
ele.send_keys("hello world")
# 设置可编辑元素的内容
ele.set_value()
# 清空可编辑元素的内容
ele.clear() 

# bool 是否展示
ele.is_dispalyed()
# bool 是否可用
ele.is_enable()
# bool 是否选中
ele.is_selected()

# 根据属性键获取属性值
ele.get_attribute("attr_key")
```



##### 元素的常用属性

```python
# 获取元素文本 
ele.text

# 获取元素坐标 ["y": 10, "x": 300]
ele.location

# 获取元素尺寸 ["width": 300, "height": 500]
ele.size

"""获取属性示例"""
# 只有通过元素才可以调用该方法，获取元素的属性以及元素的信息
book_image = driver.find_element_by_id("niv_image_book_shelf_item_grid")
print(f"书籍书籍封面 content-desc：{book_image.get_attribute('content-desc')}")
print(f"书籍书籍封面 resource-id：{book_image.get_attribute('resource-id')}")
print(f"书籍书籍封面 enabled：{book_image.get_attribute('enabled')}")
print(f"书籍书籍封面 clickable：{book_image.get_attribute('clickable')}")
print(f"书籍书籍封面 bounds：{book_image.get_attribute('bounds')}")
```



##### TouchAction

```python
from appium.webdriver.common.touch_action import TouchAction

# 短按
press(self, el=None, x=None, y=None)
# 长按 
long_press(self, el=None, x=None, y=None, duration=1000(ms))
# 点击
tap(self,el=None,x=None,y=None,count=1)
# 释放，结束屏幕上的一系列动作的命令操作
release(self)
# 移动到
move_to(self,el=None,x=None,y=None)
# 等待
wait(self,ms=0)
# 执行
perform(self)

# 组合示例 
# 点击 -> 等待 -> 移动 -> 释放 -> 执行
TouchAction(driver).press(x,y).wait(1000).move_to(x2,y2).release().perform()
```



##### MultiTouch

```Python
# 多点触控
action0 = TouchAction().tap(ele0)
action1 = TouchAction().tap(ele1)
MultiTouch().add(action0).add(action1).perform()
```



