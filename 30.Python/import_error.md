> python 3.7
>
> Mac



### 安装 psycopg2

```bash
$ pip install psycopg2
...
Error: pg_config executable not found.
...
```

出现报错：`Error: pg_config executable not found.`



**解决**

参考：https://stackoverflow.com/questions/11618898/pg-config-executable-not-found

```bash
$ which pg_config
pg_config not found

# mac
$ brew install postgresql@11
$ ll /usr/local/Cellar | grep postgresql
drwxr-xr-x  3 lan  admin    96B 12 27 12:00 postgresql@11

# pg_config 在 bin 目录下，添加到环境变量内
$ PATH="/usr/local/Cellar/postgresql@11/11.17_3/bin:$PATH"
$ pip install psycopg2
Successfully installed psycopg2-2.9.5
```

刚开始下载的 `postgresql@14` 失败，报错需要安装 python3.9，安装了 11 版本，问题解决。



### Crypto 导入失败

```python
from Crypto.Cipher import AES
# 出现 import error
```

解决

```bash
$ pip install pycryptodome
# 进入 \Lib\site-packages
# 将 crypto 首字母小写改为大写
```

