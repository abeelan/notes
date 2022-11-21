> JMESPath is a query language for JSON.



JMESPath 是 JSON 查询语言，可以从 JSON 文档中提取和转换元素。在做接口自动化测试项目时，最基础的一步就是从响应中获取各种待验证字段值，掌握 jmespath 语法，能达到事半功倍的效果。撸了一天官方文档，趁热记录下所学所得。



[Try it Out!](https://jmespath.org/ "jmespath debug")

[jmesath.py](https://github.com/jmespath/jmespath.py "jmespath-py")

[JMESPath Examples](https://jmespath.org/examples.html "Examples")



### 安装

```shell
$ pip install jemspath
```



jmespath.py 库提供了两个接口：

```python
def compile(expression):
    return parser.Parser().parse(expression)

def search(expression, data, options=None):
    return parser.Parser().parse(expression).search(data, options=options)

# options 可以通过创建实例来控制 jmespath 表达式的计算方式。
```

- compile：与 re 模块类似，使用 compile 函数编译表达式，并使用解析后的表达式执行重复搜索
- search：接收表达式和数据，返回提取结果



### 基本表达式

##### 1. 对象取值

根据键取值，如果键不存在，则返回一个 null，或该语言中与 null 等效的值。

```python
>>> from jmespath import search
>>> search("a", {"a": "foo", "b": "bar", "c": "baz"})
'foo'
>>> search("d", {"a": "foo", "b": "bar", "c": "baz"})
None
```

使用子表达式获取值，如果键不存在，则返回一个 null。

```python
>>> search("a.b.c", {"a": {"b": {"c": "value"}}})
value
>>> search("a.b.c.d", {"a": {"b": {"c": "value"}}})
None
```



##### 2. 列表取值

索引表达式，从 0 开始，索引越界，则返回 null；索引可以为负数，-1 为列表中最后一个元素。

```python
>>> search("[1]", ["a", "b", "c"])
"b"
>>> search("[3]", ["a", "b", "c"])
None
>>> search("[-1]", ["a", "b", "c"])
"c"
```



##### 3. 对象嵌套列表

```python
>>> search("a.b[0].c", {"a": {"b": [{"c": 3}, {"d": 4}]}})
3
```



##### 4. 切片

与 python 列表切片相同，[start：end：step]，留头掐尾

```python
>>> search("[0:5]", [0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
[0, 1, 2, 3, 4]

>>> search("[5:10]", [0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
[5, 6, 7, 8, 9]

# 步长为 2，为 -1 时翻转列表
>>> search("[::2]", [0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
[0, 2, 4, 6, 8]
```



##### 5. 投影

```python
# 创建数据
data = {
    "student": [
        {"name": "james", "age": 32},
        {"name": "harden", "age": 18},
        {"name": "curry", "age": 13},
        {"test": "ok"}
    ]
}
```

- 列表投影

```python
>>> search("student[*].name", data)
['james', 'harden', 'curry']
```

student 中的每个元素被收集到数组中，给到后续的表达式，这成为投影（名词来自翻译软件，我不是很懂 😂）。我理解的意思就是每一步获取到的数据结果。 通配符 `*`，表示返回列表中的全部元素。当单组元素的结果表达式为 null，则该值从收集的 **结果集** 中忽略。

所以，在最终的输出结果内没有 `{"test": "ok"}` 的值。



- 切片投影

```python
>>> search("student[:2].name", data)
['james', 'harden']
```



- 对象投影

```python
data = {
    "test": {
        "funcA": {"num": 1},
        "funcB": {"num": 2},
        "funcC": {"miss": 3},
    }
}

>>> search("test.*.num", data)
[1, 2]
```

对象投影可以分解为两个部分，左侧和右侧：

1. 左侧创建要投影的初始数组：

```
evaluate(test, inputData) -> [
	{"num": 1}, 
	{"num": 2},
    {"miss": 3}
]
```

2. 右侧作用于数组中的每个元素

```
evaluate(num, {num: 1}) -> 1
evaluate(num, {num: 2}) -> 2
evaluate(num, {miss: 3}) -> null
```

第三条不符合，最终结果过滤空值，所以最终表达式结果为 [1, 2]。



- 展平投影

列表/对象投影，在投影中创建投影时会保留原始文档的结构。

```python
data = {
  "a": [
    {
      "b": [
        {"name": "a"},
        {"name": "b"}
      ]
    },
    {
      "b": [
        {"name": "c"},
        {"name": "d"}
      ]
    }
  ]
}

>>> search("a[*].b[*].name", data)
[
    ['a', 'b'], 
    ['c', 'd']
]
```

如果只要列表下的所有值，不关心所谓的层级结构，那么就需要通过展平投影来获取结果。

```python
>>> search("a[].b[].name", data)
['a', 'b', 'c', 'd']
```

只需要将 `[*] -> []` 就可以扁平化列表。它将子列表展平到父列表，并不是递归的关系，下面例子能很好的说明。

```python
data = [
  [0, 1],
  2,
  [3],
  4,
  [5, [6, 7]]
]

# 展平一层
>>> search("[]", data)
[0, 1, 2, 3, 4, 5, [6, 7]]

# 展平第二层
>>> search("[][]", data)
[0, 1, 2, 3, 4, 5, 6, 7]
```



- 过滤投影

```python
data = {
  "book": [
    {"name": "a1", "author": "aa"},
    {"name": "a2", "author": "aa"},
    {"name": "b", "author": "bb"}
  ]
}

# 获取 aa 写的两本书
>>> search("book[?author=='aa'].name", data)
['a1', 'a2']
```

过滤器表达式是为数组定义的，一般形式为 `左侧投影 [? <表达式> <比较器> <表达式>] 右侧投影`。

条件表达式支持如下：

- `==`, tests for equality.
- `!=`, tests for inequality.
- `<`, less than.
- `<=`, less than or equal to.
- `>`, greater than.
- `>=`, greater than or equal to.



##### 6. 管道表达式

管道表达式 `<expression> | <expression>`，表示必须停止投影。将当前节点的结果传递到管道符右侧继续投影。

```python
>>> search("book[*].name | [0]", data)
"a1"

# 管道符左侧结果为 ['a1', 'a2', 'b'] 
# [0] 取该结果下标为 0 的数据，得到 a1
```



##### 7. 多选

**多选列表示例**

```python
data = {
  "people": [
    {
      "name": "a",
      "state": {"name": "up"}
    },
    {
      "name": "b",
      "state": {"name": "down"}
    },
    {
      "name": "c",
      "state": {"name": "up"}
    }
  ]
}

>>> search("people[].[name, state.name]", data)
[
    ['a', 'up'], 
    ['b', 'down'], 
    ['c', 'up']
]
```

从 json 文档中提取所需要的内容，精简结构。

`[name, state.name]` 表达式的意思是创建一个包含两个元素的列表：

- 第一个元素是计算 name 表达式得到的结果
- 第二个元素是计算 state.name 表达式的结果

因此，每个列表元素都会创建一个双元素列表，整个表达式的最终结果是一个包含两个元素列表的列表。



**多选对象示例**

与多选列表思想相同，创建的是一个散列而不是数组。

```python
>>> search("people[].{name: name, state: state.name}", data)
[
    {'name': 'a', 'state': 'up'}, 
    {'name': 'b', 'state': 'down'}, 
    {'name': 'c', 'state': 'up'}
]
```



##### 8. 函数

部分函数的使用，直接通过 test 更直观一些，见名知义，没什么好解释的。

```python
import jmespath
import pytest

class TestJmesPath:
    
    def setup_class(self):
        self.data = {
            "book": [
                {"name": "平凡的世界", "author": "路遥", "sort": 3},
                {"name": "围城", "author": "钱钟书", "sort": 2},
                {"name": "围城", "author": "钱钟书", "sort": 2},
                {"name": "活着", "author": "余华", "sort": 1},
                {"name": "麦田里的守望者", "author": "塞林格", "sort": 4},
                {"name": "挪威的森林", "author": "村上春树", "sort": 5}
            ]
        }

    def test_keys(self):
        """提取对象的 key
        注意：管道符前面是一个对象，所以需要指定下标，不能通过 * 号提取数组后获取 key，否则报错
        jmespath.exceptions.JMESPathTypeError:
            In function keys(), expected one of: ['object'], received: "array"
        """
        result = jmespath.search("book[0].test", self.data)
        assert result == ['name', 'author']

    def test_values(self):
        """提取对象的 value，不接受数组"""
        result = jmespath.search("book[0] | values(@)", self.data)
        assert result == ['平凡的世界', '路遥']

    def test_sort(self):
        """根据 sort 进行排序"""
        result = jmespath.search("book[*].sort | sort(@)", self.data)
        assert result == [1, 2, 2, 3, 4, 5]

        result = jmespath.search("book[*].author | sort(@) | [join(', ', @)]", self.data)
        assert result == ['余华, 塞林格, 村上春树, 路遥, 钱钟书, 钱钟书']

    def test_type(self):
        result = jmespath.search("book[*].sort | type(@)", self.data)
        assert result == "array"

        result = jmespath.search("book[0].name | type(@)", self.data)
        assert result == "string"

    def test_to_string(self):
        result = jmespath.search('[].to_string(@)', [1, 2, 3, "number", True])
        assert result == ["1", "2", "3", 'number', 'true']

    def test_to_number(self):
        result = jmespath.search('[].to_number(@)', ["1", "2", "3", "number", True])
        assert result == [1, 2, 3]

    def test_avg(self):
        result = jmespath.search("avg(@)", [10, 15, 20])
        assert result == 15.0

        with pytest.raises(jmespath.exceptions.JMESPathTypeError):
            jmespath.search("avg(@)", [10, False, 20])

    def test_contains(self):
        result = jmespath.search("contains(`foobar`, `foo`)", {})
        assert result is True

        result = jmespath.search("contains(`foobar`, `f123`)", {})
        assert result is False

    def test_join(self):
        # expected one of: ['array-string']
        # @ 为当前节点，得到的结果用逗号加空格分隔，然后放在当前节点下
        result = jmespath.search("join(`, `, @)", ["a", "b"])
        assert result == "a, b"

        result = jmespath.search("join(``, @)", ["a", "b"])
        assert result == "ab"

    def test_length(self):
        result = jmespath.search("length(@)", ["a", "b"])
        assert result == 2

    def test_max(self):
        result = jmespath.search("max(@)", [10, 3, 5, 5, 8])
        assert result == 10

    def test_min(self):
        result = jmespath.search("min(@)", [10, 3, 5, 5, 8])
        assert result == 3

```



##### 9. 自定义函数

接口测试项目中，有个获取列表数据后去重的需求，在官方文档上始终没有找到相关表达式或函数的使用，倒是在项目 issues 中找到这个问题，但也仅仅是一个提问...

继续查找资料，在 `jmespath.py` 项目中，找到用户自定义函数的方法，根据文档，自定义排序加去重的方法实现如下：

```python
from jmespath import search
from jmespath import functions

class CustomFunctions(functions.Functions):
    """ https://github.com/jmespath/jmespath.py
    """
    @functions.signature({'types': ['string', "array"]})
    def _func_uniq(self, arg):
        if isinstance(arg, str):
            # string of unique
            return ''.join(sorted(set(arg)))
        if isinstance(arg, list):
            # array of unique
            return sorted(set(arg))
        
options = jmespath.Options(custom_functions=CustomFunctions())

>>> search("foo.bar | uniq(@)", {'foo': {'bar': 'banana'}}, options=options)
abn

>>> search("foo.bar | uniq(@)", {'foo': {'bar': [5, 5, 2, 1]}}, options=options)
[1, 2, 5]
```



---



掌握上面的基础语法后，再回过头看官方提供的多重提取表达式示例就非常容易了，配合使用，可以满足项目中需要的大部分数据提取，特殊需求还可以通过自定义函数来实现，非常好用。

