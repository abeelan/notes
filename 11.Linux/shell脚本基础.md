### Shell

Shell 是一种应用程序，提供用户通过界面访问操作系统内核的服务。shell 脚本是为 shell 编写的脚本程序，Linux 的 shell 编程种类较多，比如：

- /bin/sh
- /bin/bash
- /bin/zsh

通常情况下，sh 和 bash 是不区分的，下文以 bash 为例。

### 命名规则

- 命名只能使用英文字母、数字和下划线，首字符不能以数字开头
- 不能包含空格
- 不能使用标点符号
- 不能使用 bash 内的关键字（可用 help 命令查看保留关键字）



### 定义与使用变量

```shell
# 定义变量
$ name="shell"
# 使用变量
$ echo hello $name
hello shell

# 定义只读变量
$ a=1
$ readonly a
$ echo $a
1
# 重新赋值就会报错
$ a=2
bash: a: readonly variable

# 删除变量
$ b=2
$ echo $b
2
$ unset b  # 删除
$ echo $b

# 无法删除只读变量
$ unset a
bash: unset: a: cannot unset: readonly variable
```

变量只作用于当前窗口，关掉窗口重新打开，变量就失效了，对只读变量同样有效。



### 变量类型

- 字符串: `a="123"`
- 拼接字符串: `b="$a, 456"`
- 数组: `values=(v1 v2 v3)`
    - 取单个值: `v2=${values[1]}`
    - 取全部值: `echo ${values[*]}` 
    - 添加元素: `values[3]=4`



### 基本运算

```shell
$ a=10
$ b=20

# 严格遵守空格
$ expr $a + $b  # 30
$ expr $a - $b  # -10
$ expr $a \* $b # 200; 乘号需要转义
$ expr $b / $a  # 2；取商
$ expr $a % $b  # 10；取余

# 两数相加其他写法
$ echo `expr $a + $b`
$ echo $(($a+$b))
$ echo $[$a+$b]

$ if [ $a == $b ]; then echo true; else echo flase; fi  # false
$ if [ $a != $b ]; then echo true; else echo flase; fi  # true
```

- `-eq`：相等
- `-ne`：不等
- `-gt`：大于
- `-lt`：小于
- `-ge`：大于等于
- `-le`：小于等于

### 控制语句

```shell
$ if condition;
> then
> 	 commands;
> elif condition;
> then
> 	 commands;
> else
> 	 commands;
> fi
```

示例：两数比较大小

```shell
$ a=1
$ b=2
$ if [ $a -eq $b ];
> then 
>   echo "equal";
> elif [ $a -lt $b ];
> then
>   echo "less than";
> else 
>   echo "greater than";
> fi
less than
```

### for 循环

```shell
# 方式一
$ for var in var1 var2 var3
> do
>  	commands
> done

# 方式二
$ for ((i=1; i<j; i++))
> do
>  	commands
> done
```

示例：逐行打印文件内容

```shell
$ for i in ${cat hotWords};
> do
>  	echo $i;
> done
```

### while 循环

```shell
$ while condition
> do
>  	commands;
> done
```

示例：递减 & 循环读取并打印内容

```shell
# 例 1: 递减
$ while (($a>=0)); 
> do 
>  	echo $a; 
>  	let "a--"; # let 计算表达式
> done

# 例 2: 循环读取文件内容并打印
# read 命令用于从终端或者文件中读取输入指令
$ while read line;
> do 
>  	echo $line;
> done < hotWords
```

### 以脚本方式运行程序

```shell
$ echo "echo demo" > demo

# 运行 sh 脚本方式一
$ sh demo
demo

# 运行方式二; 需要先添加可执行权限
$ chmod +x demo
$ ./demo
demo
```



### 脚本参数传递

- `$0` 脚本名称
- `$1~$n` 获取参数
- `$#` 传递到脚本的参数个数
- `$$` 脚本运行的进程 ID
- `$*` 以一个单字符串显示所有向脚本传递的参数
- `$?` 显示最后命令的退出状态，非 0 都表示有错误

```shell
$ cat demo
echo "脚本名称：$0"
echo "参数一值：$1"
echo "参数个数：$#"
echo "进程ID：  $$"
echo "整串参数：$*"
echo "退出状态：$?"

$ sh demo 1 2 3 4 5 6
脚本名称：demo
参数一值：1
参数个数：6
进程ID：  22173
整串参数：1 2 3 4 5 6
退出状态：0
```



### PATH 变量

- PATH 变量是一个路径列表，以 `:` 隔开；
- 如果可执行程序所在的目录在 PATH 变量路径列表里，那么输入命令时可以省略路径
- 路径列表前面的路径为优先匹配路径，匹配到即停止，可以用来实现新老版本程序的命令更换

```shell
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

# /usr/local/bin 里面的 python version = 2.7
# /usr/bin 里面的 python version = 3.6

# 会先使用路径列表前面的 python 版本
$ which python
/usr/local/bin/python  # 2.7 
```

设置环境变量，并使其立马生效；修改后如果不 source 该配置文件，则每次用户重新登录时才会自动生效。

```shell
$ vim ~/.bash_profile
$ source ~/.bash_profile
```



### 应用安装
```shell
# 1. yum: centos/redhat
$ yum search $package
$ yum install $package
$ yum remove $package

# 2. apt-get: ubuntu debian
$ apt-cache search $package
$ apt-get install $package               
$ apt-get uninstall $package

# 3. 源码编译安装：make; make install
```

##### 开源镜像站
- [华为镜像](https://mirrors.huaweicloud.com '华为镜像')
- [阿里云](https://developer.aliyun.com/mirror '阿里云')
- [豆瓣](http://pypi.douban.com/simple/ '豆瓣')

