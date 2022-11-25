> 汇总遇到的报错信息。



##### 1. ParserError: while parsing a block mapping

```bash
$ docker-compose down
ERROR: yaml.parser.ParserError: while parsing a block mapping
  in "./compose.yml", line 1, column 1
expected <block end>, but found '<block mapping start>'
  in "./compose.yml", line 25, column 3
```

原因是空格导致未对齐，检查 25 行格式错误。