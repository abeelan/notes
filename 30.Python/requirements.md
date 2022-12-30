python 项目中的依赖库，可以创建一个`requirements.txt`文件来管理。

```txt
allure-pytest=2.12.0
pytest=7.2.0
pytest-rerunfailures=10.3
pytest-sugar=0.9.6
```



```bash
# 安装
$ pip install -r requirement.txt

# 获取当前 python 环境安装包并写入文件
$ pip freeze >requirements.txt
```



```bash
# 当本地的 python 环境混用时
$ pip install pipreqs
# 进入到项目路径下，即可生成该项目的依赖包文件
$ pipreqs ./
```

