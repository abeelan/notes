> Selenium 页面消息框处理2
>
> - alert：警告消息框
> - confirm：确认消息框
> - prompt：提示消息对话框
>
> 还有一种是页面弹框，类似百度登录，这种可以直接定位到，此处忽略。



**操作 alert 的方法**

```python
# 获取当前页面上的警告框
alert = switch_to.alert()

alert.text  # 返回文本信息
alert.accept()   # 确定
alert.dismiss()  # 取消
alert.send_keys("hello") # 输入文本
```



### 1. alert

> `alert()`方法用于显示带有一条指定消息和一个 **确认** 按钮的警告框。

示例

```python
""" 
1. 切换到 iframe 内，点击按钮，弹出弹窗
2. 弹窗内点击确定或取消
3. 退出 alert，返回初始页面
"""
driver.get("https://www.runoob.com/try/try.php?filename=tryjs_alert")
driver.switch_to.frame("iframeResult")
driver.find_element(By.XPATH, '//*[@value="显示警告框"]').click()

# 切换到 alert 弹框内
alert = driver.switch_to.alert
alert.accept()  # 点击确定

# 退出弹框界面
driver.switch_to.default_content()
assert driver.find_element(By.ID, "submitBTN").text == "点击运行 》"
```



### 2. confirm

> `confirm()`方法用于显示一个带有指定消息和确认及取消按钮的对话框。
>
> 如果访问者点击"确定"，此方法返回 true，否则返回 false。

根据点击按钮不同，页面展示会有不同。

```python
""" 
1. 切换到 iframe 内，点击按钮，弹出弹窗
2. 弹窗内分别点击 确定/取消
3. 验证页面展示文本为：你按下了"确定/取消"按钮!
"""
driver.get("https://www.runoob.com/try/try.php?filename=tryjs_confirm")
driver.switch_to.frame("iframeResult")

# 切换到 alert 弹框内，点击「确定」，断言文案
driver.find_element(By.XPATH, "//body/button").click()  # 点我
alert = driver.switch_to.alert
alert.accept()  # 点击确定
assert driver.find_element(By.ID, "demo").text == '你按下了"确定"按钮!'

# 切换到 alert 弹框内，点击「取消」，断言文案
driver.find_element(By.XPATH, "//body/button").click()  # 点我
alert = driver.switch_to.alert
alert.dismiss()  # 点击取消
assert driver.find_element(By.ID, "demo").text == '你按下了"取消"按钮!'
```



### 3. prompt

> prompt()方法用于显示可提示用户进行输入的对话框。
>
> 这个方法返回用户输入的字符串。

支持用户在弹框内输入文本，用于后续处理。

```Python
"""
1. 切换到 iframe 内，点击按钮，弹出弹窗
2. 弹窗内点击取消，验证获取文本为空，文本展示元素不存在
3. 弹窗内输入文本点击确定，验证文本展示与输入一致
"""
driver.get("https://www.runoob.com/try/try.php?filename=tryjs_prompt")

# 1. 切换到 iframe 内，点击按钮，弹出弹窗
self.driver.switch_to.frame("iframeResult")
self.driver.find_element(By.XPATH, "//body/button").click()

# 2. 弹窗内点击取消，验证获取文本为空，文本展示元素不存在
alert = self.driver.switch_to.alert
alert.dismiss()
assert self.driver.find_element(By.ID, "demo").is_selected() is False

# 3. 弹窗内输入文本点击确定，验证文本展示与输入一致
self.driver.find_element(By.XPATH, "//body/button").click()
alert = self.driver.switch_to.alert
assert alert.text == "请输入你的名字"
alert.send_keys("father")
alert.accept()
assert "father" in self.driver.find_element(By.ID, "demo").text
```