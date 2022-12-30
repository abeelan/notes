iOS Predicate String 定位表达式结构：属性 + 运算符 + 值



```python
# == 运算符:
# 属性 label 的值 与 字符串 "SYSTEM(TEXT)" 相等
label == "SYSTEM(TEXT)"

# AND 运算符
# 同时满足多个条件
label == "SYSTEM(TEXT)" AND enabled == true
```

运算符



比较运算符：可以用来比较数值或字符串，`== >= <= > < != <>`



范围运算符：可用于数值和字符串的范围比对，`IN、BETWEEN`



字符串相关运算符：`CONTAINS \ BEGINSWITH \ ENDSWITH`



逻辑运算符：`AND \ OR \ NOT`



模糊匹配：`LIKE`

- 匹配一个字符`?`：`label LIKE "a?c?de"`
- 匹配多个字符`*`：`label LIKE "a*"`



正则表达式：`MATCHES`

- `label MATCHES '^a.+d$'`



### 元素属性

- type：元素类型，className
- name：元素文本内容，AccessibilityId 定位方式
- label：绝大数情况下，与 name 一致
- enabled：元素是否可点击，布尔值
- visible：元素是否可见，布尔值



### webview



网页 和 混合应用的区别就是设置 bundleID 不一致。网页应用为 safari 浏览器，混合应用为应用名称。



真机调试

手机 - 设置 - safari浏览器 - 高级 - 打开网页检查器



```bash
# 安装 ios-webkit-debug-proxy
$ ios_webkit_debug_proxy -f \ chrome-devtools://devtools/bundled/inspector.html

# 访问：http://127.0.0.1:9221/
```





