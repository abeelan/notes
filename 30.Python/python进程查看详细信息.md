> Mac
>
> Python 3.7



https://www.modb.pro/db/454999

## 安装

```bash
# 搜索仓库
$ brew search gdb

# 安装
$ brew install gdb
Error: python@3.10: the bottle needs the Apple Command Line Tools to be installed.
  You can install them, if desired, with:
    xcode-select --install
    
# 出现报错，根据报错提示安装命令行工具
$ xcode-select --install

# 重新安装后查看
$ which gdb
/usr/local/bin/gdb
$ gdb -v
GNU gdb (GDB) 12.1
```



## 创建证书

打开钥匙串应用，菜单栏点击：钥匙串访问 - 证书助理 - 创建证书

![image-20221227190532581](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221227190532581.png)

一路下一步，直到指定证书位置，选择 系统。

![image-20221227190648850](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221227190648850.png)

![image-20221227190802229](https://cdn.jsdelivr.net/gh/abeelan/image-hosting-service/img/image-20221227190802229.png)

创建完成后，还需要修改权限，钥匙串 - 系统 - 打开刚创建的证书 - 信任 - 始终信任。



```bash
# 证书授权，执行命令后输入管理员账号密码
$ codesign -fs {上面创建的证书名称} /usr/local/bin/gdb

# 关闭 MacOS 系统的 SIP 安全验证 
$ echo "set startup-with-shell off" >> ~/.gdbinit
```



```bash
$ security find-certificate -c gdb
keychain: "/Library/Keychains/System.keychain"
version: 256
class: 0x80001000
attributes:
	...
	
# 确保证书未过期
$ security find-certificate -p -c gdb | openssl x509 -checkend 0
Certificate will not expire

# 里面应该包括 Code Signing
$ security dump-trust-settings -d

$ codesign --entitlements ~/Downloads/gdb-entitlement.xml -fs gdb $(which gdb)
/usr/local/bin/gdb: replacing existing signature
```



重启电脑生效。





### 重启电脑后 python 环境不见了

```bash
$ python
dyld: Library not loaded: /usr/local/Cellar/python/3.7.3/Frameworks/Python.framework/Versions/3.7/Python
  Referenced from: /usr/local/bin/python3.10
  Reason: image not found
  
# 查看python所在的位置
$ which python
/usr/local/bin/python

# 查看 /usr/local/bin/python 的依赖
$ otool -L /usr/local/bin/python
/usr/local/bin/python:
	/usr/local/Cellar/python/3.7.3/Frameworks/Python.framework/Versions/3.7/Python (compatibility version 3.7.0, current version 3.7.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.250.1)
	
# 3.7 环境已经被干掉了，修改环境为最新的 3.10
# install_name_tool -change {old_path} {new_path} {env_path}
$ install_name_tool -change /usr/local/Cellar/python/3.7.3/Frameworks/Python.framework/Versions/3.7/Python /usr/local/Cellar/python@3.10/3.10.8/Frameworks/Python.framework/Versions/3.10/Python /usr/local/bin/python

# 正常使用
$ python
```



```bash
# 更新 pip 软链
$ rm -f /usr/local/bin/pip
$ pip3 -V
pip 22.2.2 from /usr/local/lib/python3.10/site-packages/pip (python 3.10)

$ ln -s /usr/local/Cellar/python@3.10/3.10.8/bin/pip3 /usr/local/bin/pip

$ pip -V
pip 22.2.2 from /usr/local/lib/python3.10/site-packages/pip (python 3.10)
```





### 查看 python 进程信息

重新执行程序

```bash
# 获取 python 程序进程 ID
$ ps -ef | grep python | grep -v grep | awk '{print $2}'
# 进入 gdb 交互命令
# bt 查看堆栈信息
$ gdb python 5048
(gdb)

bt    # 当前C调用栈
py-bt  # 当前Py调用栈
py-list  # 当前py代码位置
py-up  # 上一帧（py级别的帧）
py-down  # 下一帧（py级别的帧）
info thread   # 线程信息
thread <id>   # 切换到某个线程
thread apply all py-list  # 查看所有线程的py代码位置
ctrl-c  # 中断
```



上面是从网上看的，不生效

```bash
$ cd Desktop
$ gdb python
(gdb) run test.py
continue
bt
```



