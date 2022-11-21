> Linux 常用文本编辑命令



### sort

对文本内容进行排序。

常用参数：

- -b：忽略每行左侧的空白字符
- -n：按数字大小排序
- -V：按照数字版本排序
- -h：根据存储容量排序（KB、MB、GB）
- -r：倒序
- -t：指定排序后的分隔符，默认为空格
- -k：按指定的列排序
- -o：把结果保存到文件

```shell
$ sort testfile -n
$ cat testfile | sort -o test.log
```



### uniq

检查和删除文件中重复的行或列，只检查相邻行，一般与 sort 配合使用。

常用参数：

- -c：统计重复次数
- -f：跳过前 N 列
- -s：跳过前 N 个字符
- -w：仅对前 N 个字符进行比较

```shell
# 构造测试数据
$ cat testfile
tom male 18
alex male 19
ali womale 15
jerry male 18
kd male 18

# 根据同岁人口的数量排行
$ cat testfile | sort -k 3 -n | uniq -c -f 2 | sort -nr
  3 jerry male 18
  1 ali womale 15
  1 alex male 19
```



### wc

字符统计。

常用参数：

- -c：统计字节数
- -l：统计行数
- -w：统计单词数

```shell
$ cat testfile
hello world
tester
gogogo!
\n
\n

# 行数、单词数、字符数（换行符、空格也算字符）
$ cat testfile | wc
	5       4      29
```



### grep

基于正则表达式查找满足条件的行或者字符串。

**内容检索：**

- -o：根据正则仅提取内容
- -A1：取匹配到内容的后一行

- -B2：取匹配到内容的前两行

```shell
$ grep go testfile
gogogo!

$ grep "[a-z]*er " testfile
tester lan

$ grep -o "[a-z]*er " -A1 testfile
tester
gogogo!
```

**文件检索：**

- -r：递归查找目录
- -h：仅展示匹配的内容，不展示匹配的文件名，默认是展示的（-H）
- -l：仅展示匹配的文件名，不展示匹配的内容

```shell
$ grep -r hello tmp
tmp/testfile:hello world

$ grep -r -h hello tmp
hello world

$ grep -r -l hello tmp
tmp/testfile
```

**范围约束：**

- -i：忽略大小写
- -v：不显示匹配的行
- -E：使用扩展正则表达式
- --include：文件/目录范围约束，仅在匹配的文件内查找内容

```shell
# 默认区分大小写
$ grep HELLO testfile

# -i 忽略大小写
$ grep -i HELLO testfile
hello world

# -v 除匹配的行外都显示，空行也显示
$ grep -v -r hello testfile
testfile:tester lan
testfile:gogogo!
testfile:
testfile:

# -E 写正则表达式可以不使用转义符
# ^$ 代表空行，所以这里输出了两个空行
$ grep -E "^$" testfile


$ grep -r test tmp --include "test*"
tmp/testfile:tester lan
```



通过 grep 过滤进程时，由于 grep 本身会开启新进程，所以通过 -v 参数过滤匹配行，才是真正想要的内容。

```shell
$ ps -ef | grep ssh
 501 35036     1   0 301121   ??       0:00.19 /usr/bin/ssh-agent -l
 501 57090 49928   0 12:20上午 ttys000  0:00.00 grep ssh
 
$ ps -ef | grep ssh | grep -v grep
 501 35036     1   0 301121  ??  0:00.19 /usr/bin/ssh-agent -l
```



### awk

根据定位到的数据处理其中的分段。

awk 具备完整的编程特性，比如执行命令，网络请求等。核心语法：

```shell
# 匹配表达式 + 行为表达式
$ awk 'pattern{action}'
```



**上下文变量**

- 开始 BEGIN 结束 END
- 行数 NR
- 字段与字段数 $1 $2 .. $N $NF
- 整行 $0
- 字段分隔符 FS，简写 -F
- 输出数据的字段分隔符 OFS
- 记录分隔符 RS
- 输出字段的行分隔符 ORS

```shell
# 通过逗号分隔取第一二列
$ echo '1, 2, 3' | awk -F , '{print $1, $2}'
1  2
```



**字段变量用法**

- -F：参数指定字段分隔符，可以用 ｜ 指定多个分隔符，例：`-F '<|>'`
- BEGIN{FS="_"} 也可以表示分隔符
- $0 表示当前的记录
- $1 表示第一个字段
- $N 表示第 N 个字段
- $NF 表示最后一个字段
- $(NF-1) 表示倒数第二个字段



**pattern 表达式**

- 正则匹配
    - 根据字段匹配：$1~/pattern/
    - 整行匹配简写：/pattern/
- 比较表达式
    - $2>2 
    - $1=="b"

```shell
# 获取第三行的内容
$ echo '1
> 2
> 3' | awk "NR==3"
3

# 正则匹配 3
# 默认 action 为 {print $0}
# 下面的 awk 等价于 awk '$1~/3/{print $0}'
$ echo '1
> 2
> 3' | awk '$1~/3/'
```

匹配表达式案例

```shell
# 开始和结束
$ awk 'BEGIN{}END{}'

# 正则：整行匹配
$ awk '/test/'
# 正则：字段匹配
$ awk '$2~/test/'

# 取第二行
$ awk 'NR==2'
# 去掉第一行
$ awk 'NR>1'

# 区间选则，包含开始和结束行，中间以逗号隔开
$ awk '/aa/, /bb/'
$ awk '/1/, NR==2'
```



**action 表达式**

如果行为表达式前面没有 pattern 表达式，则代表对所有数据生效。

- 打印：{print $2}
- 赋值：{$1="abc"}
- 处理函数
- 原始内容：$0
- 更新后内容：{$1=$1;print $0}

```shell
# 单行转多行
$ echo 1:2:3 | awk 'BEGIN{RS=":"}{print $0}'
1
2
3

# 多行转单行
$ echo '1
> 2
> 3' | awk 'BEGIN{RS="";FS="\n";OFS=":"}{$1=$1;print $0}'
1:2:3

# 多行转多行方法二
$ echo '1
> 2
> 3' | awk 'BEGIN{RS="";FS="\n";OFS=":"}{$1=$1;print $0}'
1:2:3

# 求第二列平均数
$ echo '1,10
> 2,20
> 3,30' | awk 'BEGIN{total=0;FS=","}{total+=$2}END{print total/NR}'
20
```



**词典/array 结构**

```shell
# 求第三列总数
$ echo 'a, 1, 10
> a, 2, 20
> b, 1, 30
> b, 2, 40' | awk '{data[$1]+=$3} END{for(k in data) print k, data[k]}'
a, 30
b, 70

# 求第三列平均数
$ echo 'a, 1, 10
a, 2, 20
b, 1, 30
b, 2, 40' | awk '{data[$1]+=$3; count[$1]+=1} END{for(k in data) print k, data[k]/count[k]}'
a, 15
b, 35
```



### sed

流式编译器，定位并修改数据。

基本语法：sed [addr]X[options]

常用参数：

- -e：根据表达式处理文本
- -n：仅展示处理后的结果
- -i：直接修改源文件
- -E：扩展表达式
- -debug：调试模式



**pattern 表达式**

- 行数 20
- 行数范围 20,40
- 正则匹配 /pattern/
- 区间匹配 //,//

```shell
$ sed -n '1p' testfile
hello world

$ sed -n '2,3p' testfile
tester lan
gogogo!

$ sed -n '/world/p' testfile
hello world
```



**action 表达式**

- p 打印，结合 -n 参数使用
- s 查找替换
- d 删除
- a 在内容匹配行之后追加
- c 取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行
- i 插入内容到匹配行之前
- e 执行命令
- 分组匹配与字段提取

```shell
# 删除最后一行, $ 在这里不是正则，代表最后一行
$ sed '$d' testfile

# 删除多行
$ sed '1,3d' testfile
```

s 表达式

```shell
# 替换
$ echo 'good night' | sed 's/night/evening/'
good evening

# / 可以用任意符号代替，只要三个符号是一致的就行
$ echo 'good night' | sed 's#night#evening!#'
good evening!

# 默认只会替换第一个符合表达式的内容
$ echo 'good night' | sed 's#o#_#'
g_od night

# 通过 g 参数替换全局
$ echo 'good night' | sed 's#o#_#g'
g__d night

# 通过 & 匹配内容，n 前面添加 123
$ echo 'good night' | sed 's/n/123&/'
good 123night

# n 后面添加 123
$ echo 'good night' | sed 's/n/&123/'
good n123ight
```

反向引用

- 使用 () 对数据进行分组
- 使用 `\3 \2 \1` 反向引用分组

```shell
# 将 123 倒序替换
$ echo 0 1 2 3 4 5 | sed -E 's#([1-3]) ([1-3]) ([1-3])#\3 \2 \1#'
0 3 2 1 4 5
```

