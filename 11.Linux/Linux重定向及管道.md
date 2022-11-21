### 输入输出重定向

- 标准输入（0）
- 标准输出（1）
- 错误输出（2）

程序接收用户标准输入，根据输入的指令执行程序，执行完成后进行标准输出，当程序异常时，会有错误输出。



输入重定向：把文件导入到命令中

输出重定向：把命令输出的信息导入到文件中

- 标准输出重定向
- 错误输出重定向
- 清空写入模式
- 追加写入模式

```shell
# 将标准输出重定向到文件内
$ echo 111 > log

# 将标准输入重定向到指定文件
# 控制台就不会接收键盘指令了，直接从文件读取输入
$ read a < log

# 将内容标准输出
$ echo $a
111

# 错误输出；123 为不存在的目录
# 错误信息并没有重定向到文件内，而是在控制台输出了
$ ls 123 > error
ls: 123: No such file or directory

# 默认为标准输出，不需要添加文件描述符 1
# 错误输出需要添加文件描述符 2 才可以重定向到文件
$ ls 123 2> error
$ cat error
ls: 123: No such file or directory

# 将错误输出与标准输出共同写入到文件
$ (echo 123; ls 456) > error 2>&1
$ cat error
123
ls: 456: No such file or directory
```

以上都为清空写入模式，将输出重定向到一个新的文件，下面是追加写入模式的例子。

```shell
$ echo "add test 1" >> test
$ echo "add test 2" >> test
$ cat test
add test 1
add test 2
```



### 管道

重定向的作用就是将命令与文件连接，Linux 还有一种功能将命令与命令连接，也就是把前一个命令的输出作为后一个命令的输入，以这种方式连接形成了 **管道**，管道符为 `｜`。

```shell
# 将第一个命令的标准输出作为第二个命令的标准输入
# {} 代表代码块
$ echo world | { read line; echo hello $line!; }
hello world!
```



**管道执行的上下文控制**

```shell
$ echo hello world | read x; echo $x

$ echo $x

```

通过这个方式无法直接输出变量 x，因为通过管道符连接 `echo hello world | read x;` 两个命令后，后者是通过子进程的方式运行的，执行完后就销毁，所以是打印不到变量 x 的。

1. 通过代码块连接方式执行；
2. 使用逻辑控制 `while read` 组合；

```shell
$ echo hello world | { read x; echo $x;}
hello world

$ echo hello world | while read x; do echo $x; done
hello world
```



**Linux 三剑客**

> grep awk sed

https://mp.weixin.qq.com/s/DUB7k7jw_wP-nwL7mxUe_w

示例：获取到用户搜索的热词后，根据分数进行排序，查找到最热门的 3 个搜索词。

```json
// 测试数据; filename => hotWords
{
    "hotWords": [
        {"gid": 0,"score": 5562,"word": "现代都市","type": 1},
        {"gid": 0,"score": 5401,"word": "玄幻奇幻","type": 1},
        {"gid": 0,"score": 5025,"word": "宅小说","type": 1},
        {"gid": 0,"score": 4091,"word": "历史军事","type": 1},
        {"gid": 0,"score": 2666,"word": "科幻小说","type": 1}
}
```

组合命令如下：

```shell
$ cat hotWords \
| grep -o 'gid[^}]*}' \
| awk -F '"' '{print $4, $7}' \
| awk 'BEGIN{RS=":"; FS=","; OFS=":"}{$1=$1; print $0}' \
| sed 's/ *//' \
| grep -v '^$' \
| sort -t ':' -k2 -nr \
| head -n 3
5562: 现代都市
5401: 玄幻奇幻
5025: 宅小说
```

- grep -o：过滤出数据行
- awk -F：过滤出数据列
- awk BEGIN：格式化数据
- sed：去除首行空格
- grep -v：获取除空行外的数据
- sort：按照数字列进行排序，反序
- head -n 3：取前三列数据



### 正则表达式

基础正则：

- `^` 开头
- `$` 结尾
- `[]` 表示区间，`[^}]` 表示除大括号之外的数据，到大括号就停止查找
- `*` 表示 0 个或多个
- `.` 表示任意字符

扩展正则：

- `?` 非贪婪匹配
- `+` 一个或者多个
- `()` 分组
- `{}` 范围约束
- `|` 匹配多个表达式中的任何一个，`1|3`