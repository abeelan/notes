> else 使用汇总。



## 问题

阅读别人代码，有点疑惑，精简后如下：

```
def code_example(arg=None):
    for i in range(5):
        if arg:
            break
    else:
        print('else branch')
```

循环语句后面直接跟了 else 语句，未报错，程序正常运行。一般 else 都是配合判断语句用，**那么这里的 else 是什么作用呢**？

**尝试**

```
for i in range(2):
    print(i)
else:
    print("else branch")

>>>
0
1
else branch
```

test01：根据打印信息发现，for 循环正常执行完成后执行了 else 分支；

```
for i in range(2):
    continue
else:
    print("else branch")

>>>
else branch
```

test02：循环体内增加 continue 跳出，执行完成循环后，正常执行 else 分支；

```
for i in range(2):
    break
else:
    print("else branch")

>>>
```

test03：如果 break 掉循环，打断循环，没有任何输出，也就是不走 else 分支；

```
def test():
    for i in range(2):
        return
    else:
        print("else branch")

>>>
```

test04：尝试 return 语句，打断循环，也是不走 else 分支。

**结论**
for … else …

- 仅当循环体全部执行完成，才执行 else 分支；
- 当循环过程被打断，则不执行 else 分支。

------

**扩展**
Python 支持 else 语句汇总：

- **for … else …**
- **while … else …**
- **try … except … else …**
- **if … elif … else …**

**while**
与 for 循环相同步骤测试，结论一样

**try**

1. 当 try 内无异常执行完成后，执行 else 分支；
2. 当 try 内出现异常，执行到 except，不再执行 else 分支。

```
def test_01():
    try:
        print("try")
    except:
        print("except")
    else:
        print("else")

>>>
try
else
-----------------------
def test02():
    try:
        5 / 0
    except:
        print("except")
    else:
        print("else")

>>>
except
```

## 总结

- for、while 循环
  当循环语句全部正常执行完成(包括 continue)，会继续执行 else 分支；当循环语句被打断(break\return)，不再执行 else 分支
- try 异常处理
  当 try 语句无异常执行完成时，会继续执行 else 分支；当抛出异常后，不再执行 else 分支
- if 条件判断
  不符合 if 或者 elif，才执行 else 分支

