

> allure python 使用方法



生成测试报告不启动服务

```bash
$ allure generate {allure_result}

# -o 指定生成报告目录
# --clean 在生成新的 Allure 报告目录之前，清除该目录
$ allure generate {allure_result} -o {allure_report} --clean
```



启动服务，展示报告。

```bash
$ allure open {allure-report}
```





生成测试报告并启动服务，在网页上展示。相当于自动 generate + open

```bash
$ allure serve {result_path}
```

