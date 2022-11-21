> 三剑客实战练习

⚠️ 用法 --- 填写文章连接



##### 1、统计：查出日志中包含 404、500 的报错行数

```shell
# 关键字前后加空格会更佳精准过滤
$ cat tmp.log \
> | grep -E ' 404 | 500 ' \
> | wc -l
4

# ~ 与 !~ 代表是否匹配正则
$ awk '$9~/404|500/' tmp.log \
> | wc -l
4
```



##### 2、找出 500 错误行的上下文请求

```shell
# A=after B=before；上下各两行
$ cat tmp.log \
> | grep -E '500' -B2 -A2
```



##### 3、统计：找出访问量前三高的 IP 地址

```shell
# 请求方 IP 地址在第一列
$ awk '{print $1}' tmp.log \
> | sort | uniq -c | sort -nr \
> | head -3
```



##### 4、统计：根据路径匹配获取请求数并排行

```shell
# tmp.log (简化日志)
"POST /demo/123?do=1 HTTP/2.0" 200 367 "-" "Android 10.0;" 127.0.0.1:8000
"POST /demo/456?do=2 HTTP/2.0" 200 367 "-" "Android 10.0;" 127.0.0.1:8000
"POST /demo/123/test/456?go=1 HTTP/2.0" 200 367 "-" "Android 10.0;" 127.0.0.1:8000

$ grep -E ' /demo/[0-9]{1,}' tmp1.log \
> | awk '{print $2}' 
> | sed -E 's#/demo/[0-9]*[?].*#/demo/:int:#' \
> | sed -E 's#/demo/[0-9]*/test/[0-9]*[?].*#/demo/:int:/test#' \
> | sort | uniq -c | sort -nr | head -2
 2 /demo/:int:
 1 /demo/:int:/test
```



##### 5、查看当前开放的端口号和进程

```shell
$ netstat -ntpl | sed 1,2d | awk '{print $1, $4, $7}'
tcp 127.0.0.1:199 1002/snmpd
tcp 0.0.0.0:20755 26345/sshd
```



##### 7、统计当前机器连接数

```shell
$ netstat -tnp \
| sed 1,2d \
| awk '{print $5}' \
| awk -F ':' '{print $1}' \
| sort | uniq -c | sort -nr 
```



##### 8、统计一个进程的 CPU 和内存信息

```shell
# 使用 PS 命令统计并不准，占用内存 / 进程运行总时间
# 所以运行时间越久，CPU 占用更小，无法统计当前的真实使用率占比
$ for i in {1..20}; \
> do 
>   sleep 1; 
>   ps -o %cpu %mem -p {pid}; 
> done

# 使用 top 命令；更准确也更耗内存
# 统计 20 次，并在统计完成后打印出平均值
$ top -b -p {pid} -n 20 -d 1 \
> | grep --line-buffered {pid} \  # 逐行显示
> | awk 'BEGIN{print "cpu mem"}{print $9, $10; c+=$9; m+=$10}END{print "--------"; print c/NR, m/NR}'

# top 命令实时统计平均值; 并绘制图表
$ top -b -p {pid} -n 20 -d 1 \
> | grep --line-buffered {pid} \
> | awk 'BEGIN{OFS="\t"; print "cpu", "mem", "avgc", "avgm"}{c+=$9; m+=$10; print $9, $10, c/NR, m/NR}' \
> | gnuplot -e "set term du; plot '<cat' using 1 with lines"

# 直接在窗口展示
$ gnuplot -e "set term du; plot '<cat' using 1 with lines"
# 生成为图片
$ gnuplot -e "set term png; plot '<cat' using 1 with lines" >tmp/top.png
```

