# Jenkins Pipeline

### 一、概念

将整个持续集成的过程用解释性代码 `jenkinsfile` 来描述。

### 二、Jenkinsfile 使用方法

- Pipeline script
- Pipeline script from SCM

##### 1、Pipeline script

创建第一个 pipeline 项目

- 新建项目
- 输入项目名称
- 选择流水线项目
- 点击确定
- 进入配置页面，配置 pipeline

下拉到页面最底部，定义流水线脚本有两种方式，默认是在配置页面编辑脚本，也就是 `Pipeline script`，点击输入框右上角的

`try simple Pipeline...` 

就可以自动生成一个 demo 脚本。

```json
pipeline {
    // 构建节点，any 代表任意节点
    agent any  
	  // 构建阶段，可以有多个阶段
    stages {  
        // 构建阶段一
        stage("Hello") {
            // 阶段一执行的步骤一
            // 可以有多个步骤
            steps {
                // 步骤内构建脚本
                echo "Hello World!"
            }
        }
    }
    // 构建后置操作
    post {
        // always 代表总是触发
        always {  
            echo "Say goodbye!"
        }
    }
}
```

保存配置后，进行构建，第一个流水线任务就建立完成了。

![效果图](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211203184523509.png)



##### 2、Pipeline script from SCM

远程仓库检出脚本构建

在配置页面上编写 jenkinsfile 有个缺点，如果内容有变更的话，需要登录 Jenkins 平台修改，比较麻烦。所以流水线项目提供从远程仓库检出 jenkiinsfile 的方式构建。

- 新建一个 Jenkins file，填入上面的脚本内容，提交远程代码仓库。
- 回到配置页面，流水线定义选中 `Pipeline script form SCM`，配置远程代码仓库
- 选择 SVN（Git 类似）
- 填写仓库地址（Repository URL）
- 配置验证信息，有代码仓库访问权限的账号（Credentials）
- 填写 Jenkinsfile 脚本路径，就是远程仓库脚本的存放目录
- eg： `./ci/jenkinsfile`
- 保存后进行构建

![从 SVN 检出进行构建](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20211203184322389.png)

### 三、Jenkinsfile 语法

- Declarative pipeline（结构化方式）
- Scripts pipeline（groovy 语法）

##### Declarative pipeline

1、所有内容必须包含在一个 pipeline 代码块内：`pipeline {}`

```json
pipeline {
	...
}
```

2、agent（必填指令）：执行节点

- any：可以在任意节点上执行 pipeline 的构建
- none：pipeline 不分配全局节点，每个 stage 自己分配
- label：指定运行节点的 label
- node：自定义运行节点配置
    - 指定 label
    - 指定 自定义工作目录
- docker：控制目标节点上的 docker 运行相关内容

```json
pipeline {
  agent {
    node {
      label "myslave"
      customWorkspace "myWorkSpace"
    }
  }
}
```

3、stages（必填指令）：包含一个或者多个 stage 的序列，pipeline 大部分工作在此执行。

- 无参数
- 唯一

4、stage（必填指令）：包含在 stages 中，pipeline 完成的所有工作都需要包含到 stage 中

- 无参数
- 需要定义 stage 名字
- 可以写多个

5、steps（必填指令）：包含在 stage 中，具体执行步骤

- 无参数
- 可以写多个

6、post（非必填指令）：定义 stages 运行结束后的操作

- always：无论 stages 运行结果如何，都会执行
- changed：只有当运行结果与上次运行结果不同时，才会执行
- failure：仅当运行结果为失败时，才会执行
- sucess：仅当运行结果为成功时，才会执行
- unstable：仅当运行结果不稳定时，才会执行
- aborted：仅当运行中止状态时，才会执行

```json
post {
  always {
    echo "always"
  }
  success {
    echo "success"
    sleep 2
  }
  unstable {
    echo "unstable"
  }
}
```

7、environment（非必填指令）：定义运行时环境变量，无参数

```json
environment {
  test = 'hello world'
}

stages {
  stage("print environment") {
    steps {
      echo test
		}
	}
}
```

8、options（非必填指令）：定义 pipeline 的专有属性，配置页内的其他自定义项

- buildDiscarder：保持构建的最大个数
- disabelConcurrentBuilds：不允许并行执行 pipeline 任务
- timeout：pipeline 超时时间
- retry：重试次数
- timestamps：预定义由 pipeline 生成的所有控制台输出时间
- skipStagesAfterUnstable：一旦构建进入不稳定状态，就跳过此 stage

```js
options {
	timeout(time: 30, unit: "SECONDS")
	buildDiscarder(logRotator(numToKeepStr: '2'))
	retry(5)
}
```

9、parameters（非必填指令）：定义 pipeline 的专有参数列表

- 支持的数据类型
    - booleanParam
    - choice
    - credentials：证书
    - file
    - text
    - password
    - run
    - string
- 类似参数化构建的选项

```js
parameters {
	string(name: "PERSON", defaultValue: "jenkins", description: "输入的文本参数")
}
stages {
	stage("Test Parameters") {
		steps {
			echo "Hello, ${params.PERSON}!"
		}
	}
}
```

10、triggers（非必填指令）：定义了 piplien 自动化触发的方式

- cron：接收一个 cron 风格的字符串来定义 pipeline 触发的常规间隔
- pollSCM：接收一个 cron 风格的字符串来定义 Jenjkins 检查 SCM 源更改的常规间隔；如果存在更改，则 pipeline 被重新触发。

```js
triggers {
	pollSCM('H */4 * * 1-5')
}
```

11、parallel（非必填指令）：并行构建

```js
pipeline {
  agent none
  stages {
    stage("并行构建") {
      parallel {
        stage("stage - 001") {
          agent {
            label "slave1"
          }
          steps {
            echo "在 slave1 节点上执行 stage - 001."
            sh "ifconfig"
            sleep 10
          }
        }
        stage("stage -002") {
          agent {
            label "slave2"
          }
          steps {
            echo "在 slave2 节点上执行 stage - 002."
            sh "ifconfig"
            sleep 10
          }
        }
      }
    }
  }
}
```



##### 2、Scripts pipeline

基于 groovy 语法定制的一种领域专用语言（DSL）

> DSL (Domain Specific Language)
>
> A specialized computer language designed for a specific task.
>
> 为了解决某一类任务而专门设计的计算机语言。



优点：

- 灵活性高
- 可扩展性更好
- 与 Declarative pipeline 语法类型



流程控制 - if/else

```js
node {
	stage("if/else demo"){
		if (env.BRANCH_NAME == "master") {
			echo "master branch"
		} else {
			echo "else branch"
		}
	}
}
```



流程控制 - try/carch

```js
node {
	echo "This is test stage which run on the slave agent."
	try {
		echo "try block."
	} catch (exc) {
		echo "catch block."
	} finally {
		// 兜底执行
		echo "finally block."
	}
}
```



控制全局工具设置的环境变量与运行命令

- 定义 maven Name & MAVEN_HOME
- 定义 java Name & JAVA_HOME
- 定义 allure Name & ALLURE_HOME
- ...

```js
stage("maven deploy") {
	node("master") {
		sh ". ~/.bash_profile"
		
		// 定义 maven 工具，工具在全局工具设置添加好，这里去获取
		// 全局工具设置（Manager Jenkins - Global Tools Configuration）
		def mvnHome = tool "maven_3.6.0_master"
		def jdkHome = tool "jdk_1.8_master"
		env.PATH = "${mvnHome}/bin:${env.PATH}"
		env.PATH = "${jdkHome}/bin:${env.PATH}"
		sh "mvn clean install"
		sh "mv target/iWeb.war target/ROOT.war"
	}
}
```

以上就是 jenkinsfile 的基本用法，先有一个大概的认识，了解了解语法，编写过程中随用随查。
