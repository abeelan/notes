> 记录一个 pytest-reruns 插件和 pytest-timeout 插件的 [兼容性问题](https://github.com/pytest-dev/pytest-rerunfailures/issues/99 "issues-99")。

先说结论：
当两个插件一起使用时，需要设置 `timeout_func_only=true`



##### 问题

接口自动化项目在容器内执行时 `reruns` 插件闪退，配置的 `timeout` 时间不生效，导致项目一直在阻塞状态，本文记录下该问题。

看看这个项目构建时长，受得了吗！？

![](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221229161622651.png)




##### 复现
编写测试用例

```python
import pytest
import time

def test_rerun():
    for i in range(5):
        time.sleep(1)
		    print(f"sleep {i + 1}")
    assert 0
```

验证插件各自都正常工作

```shell
$ pytest test.py -raR --reruns 2

Results (15.14s):
       1 failed
         - test.py:14 test_rerun
       2 rerun
       
$ pytest test.py -raR --timeout 3

E     Failed: Timeout >3.0s
```

两个插件结合使用
```shell
$ pytest test.py -sraR \
  --reruns 2 --reruns-delay=2 \
  --timeout==3
  
timeout: 3.0s
timeout method: signal
timeout func_only: False
sleep 1
sleep 2

sleep 1
sleep 2
sleep 3
sleep 4
sleep 5

sleep 1
sleep 2
sleep 3
sleep 4
sleep 5
```
可以看到重试用例后，超时并没有生效，依然等待 `5` 秒。

##### 解决

官方仓库内找到解决办法，需要增加一个配置项。

```bash
# -o 代表覆盖 pytest.ini 内定义的配置项	
$ pytest test.py -sraR \
  --reruns 2 --reruns-delay=2 \
  --timeout=3 \
  -o timeout_func_only=true  # 需增加配置项

timeout=3 
timeout: 3.0s
timeout method: signal
timeout func_only: True
sleep 1
sleep 2

sleep 1
sleep 2

sleep 1
sleep 2
```
或者在配置文件内增加
```ini
[pytest]
timeout = 5
timeout_func_only=true
```