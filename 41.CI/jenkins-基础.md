持续集成测试流程记录。



[jenkins_pipeline_快速入门 38](https://pdf.ceshiren.com/ck18/jenkins_pipeline_快速入门.html)

[jenkins_pipeline_语法详解 19](https://pdf.ceshiren.com/ck18/jenkins_pipeline_语法详解.html)

[jenkins_pipeline_Sharedlib 8](https://pdf.ceshiren.com/ck18/jenkins_pipeline_Sharedlib.html)

[jenkins_pipeline与k8s集成 8](https://pdf.ceshiren.com/ck18/jenkins_pipeline_pipeline与k8s_集成.html)

[jenkins_pipeline_jenkins集成k8s的原理和配置总结 8](https://pdf.ceshiren.com/ck18/jenkins_pipeline_jenkins集成k8s的原理和配置总结.html)

[jenkins_pipeline_多分支pipeline 10](https://pdf.ceshiren.com/ck18/jenkins_pipeline_多分支pipeline.html)



### jenkins 搭建

```shell
# 创建或者映射 docker 文件映射卷；二选一
$ docker volume create jenkins   # 创建新的映射卷
$ docker volume inspect jenkins  # 本地文件映射
# 创建容器;lts 长期维护版本，稳定版本
$ docker run -d --name jenkins -p 8080:8080 -p 5000:5000 -v {本地目录}/jenkins:/var/jenkins_home jenkins/jenkins:lts
# 获得管理员密码
$ docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```



### 插件

可以先配置代理，然后再下载插件。

- email extension
- email extension template
- blue ocean



### job 管理

##### 修改系统配置

默认shell：bash

默认邮箱：邮箱地址与账户

默认地址：服务器域名

安全：设置安全制度

时区：时区修改

插件：设置代理、安装插件、更新插件

##### 节点管理

1. jenkins 的任务可以分布在不同的节点上运行

2. 节点上需要配置 java 运行环境：Java version > 1.5

3. 节点支持 windows、linux、mac

4. jenkins 运行的主机在逻辑上是 master 节点

设置：Manage Jenkins --> System configuration --> mange Nodes and Clouds --> New Node 

节点连接方式：

- 8080 端口：jenkins 服务器对外 URL 地址
- 50000 端口：slave 节点与 Jenkins 的通讯端口



##### 用户权限控制

- 初始化过程会先注册一个管理员用户
- 管理员用户登录后再创建后续的一般用户
- 启用用户安全配置：Manage Jenkins --> Configure Global Security



##### 报警机制

```jenkinsfile
# 模版配置
# 通过 Jenkins 的参数定制自己的 Email 模版
# 常用的 key 值
$BUILD_STATUS:构建结果
$PROJECT_NAME:构建脚本名称
$BUILD_NUMBER:构建脚本编号
$JOB_DESCRIPTION:构建项目描述
$CAUSE:脚本启动原因
$BUILD_URL:脚本构建详情URL地址
```

用户在 jenkins 构建任何之后发送 email 通知

- 需要下载插件：Email Extension、Email Extension Template
- 配置邮件发送规则
- 配置邮件模版

在系统设置内，设置管理员邮箱地址。

https://blog.csdn.net/weixin_35688430/article/details/119599712



##### 父子 job

父子多任务运行，任务启动的触发条件，是其他任务的运行结果，有以下触发条件：

- 前驱任务成功
- 前驱任务失败
- 前驱任务不稳定

使用场景：有先后次序关系的任务，比如：部署环境 -> 验收测试任务



### cli 命令行

需要本地先下载 jenkins-cli.jar

Manage Jenkins - Jenkins CLI，然后点击下载链接进行下载。

```shell
# 执行重启命令
$ java -jar jenkins-cli.jar -s http://localhost:8081/jenkins/ restart
```

ERROR: anonymous is missing the Overall/Administer permission

```shell
# 加上 -auth 管理员账号就可以了
$ java -jar jenkins-cli.jar -s http://localhost:8081/jenkins/ -auth admin:password restart
```

