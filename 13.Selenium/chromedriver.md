批量杀死 chromedriver 进程

```bash
$ ps -ef \
| grep chromedriver | grep -v grep \
| awk '{print $2}' \
| awk 'BEGIN{RS="";FS="\n";OFS=" "}{$1=$1;print $0}' \
| xargs kill -9
```

