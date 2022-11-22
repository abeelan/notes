### Gogo - 一款极易搭建的自助 Git 服务。

官网：https://gogs.io/

地址：http://gogs.ieasou.cn/

账号：lanyangbi｜lanyangbi@6666



### CI 平台

官网：https://www.drone.io/

地址：http://drone2.ieasou.cn/

账号：同上



### 私有镜像仓库

地址：http://hub.evbj.easou.com

账号：查看 .drone.yml 配置文件



### Lens - Kubernetes runs Desktop

下载地址（Mac）：https://k8slens.dev

![image-20220113174858897](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220113174858897.png)

配置文件找管理员生成，填入即可。



查看当前运行的服务

![image-20220113175035622](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220113175035622.png)

查看服务访问地址

![image-20220113175116393](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220113175116393.png)



### 搭建 Nextcloud

1、在 gogs 上先创建 nextcloud-mariadb 项目，然后需要写入四个文件：

- .drone.yml：流水想项目配置文件
- Dockerfile：项目镜像构建
- deploy-tmp.yaml：项目部署配置文件
- README.md：非必需，一般用来写项目介绍



```yaml
# .drone.yml
kind: pipeline
type: kubernetes
name: nextcloud-mariadb  # 【QA】项目名称

clone:
  disable: true

metadata:
  namespace: drone

steps:
 - name: clone
   image: hub.evbj.easou.com/dev/alpine-git:20200622
   commands:
   - git clone $DRONE_GIT_HTTP_URL .

 - name: build
   image: plugins/docker
   settings:
     username: admin
     password: Easou2)1*
     insecure: true
     mirror: https://ci7pm4nx.mirror.aliyuncs.com
     registry: hub.evbj.easou.com
     repo: hub.evbj.easou.com/tech-qa/${DRONE_REPO_NAME}  # 【QA】repo 路径修改为 tech-qa
     tag: v1.0.0  # 【QA】版本号可自定义

 - name: deployment
   image: hub.evbj.easou.com/dev/drone-agent:v2.0.0
   pull: always
   # privileged: true
   environment:
     JNLP_ENV: tech-qa
     JNLP_REPLICAS: 1
     JNLP_TAG: v1.0.0  # 【QA】版本号可自定义
     JNLP_VERSION: v1  # default v1, v2 for canary
     DEPLOY_ENV: k8s-2 # 可以选择把应用部署到集群:k8s-1 or k8s-2
     JNLP_REPO: hub.evbj.easou.com
     JNLP_CONTAINER_PORT: 3306  # 【QA】端口号需要修改
     JNLP_INGRESS: nginx
     JNLP_SVC_MODE: http
     JNLP_STORAGE_CLASS: rbd
     JNLP_DOMAIN: .ieasou.cn
     JNLP_CONTROL: StatefulSet
     JNLP_STORAGE_CAPACITY: 20Gi
     JNLP_MOUNT_PATH: /data
     JNLP_LIVENESS_INIT: 30
     JNLP_LIVENESS_PER: 15
     JNLP_READINESS_INIT: 30
     JNLP_READINESS_PER: 15
     JNLP_INGRESS_PUB: yes
   commands:
     # 切换目标集群
     - kubecm s $DEPLOY_ENV
     # 生成配置YAML文件
     - python3 /root/tools.py -b
     # 部署服务到kubernetes上
     - python3 /root/tools.py -a
     # 检查服务部署状态
     - python3 /root/tools.py -c
     # 添加dns解析记录,生成访问域名
     - python3 /root/tools.py -d

 - name: success
   image: hub.evbj.easou.com/dev/drone-wechat:20200622
   settings:
     corpid: ww419ee4063735e1c0
     corp_secret: zpiRBLETH9eLwIMQ4eJ_r_dcm3BPSGeLHvTcft8Ot-M
     agent_id: 1000004
     title: "Pipeline ${DRONE_REPO_NAME} Success"
     description: "${DRONE_BUILD_LINK} 部署完成"
     msg_url: ${DRONE_BUILD_LINK}
     btn_txt: "否"
   when:
     status:
     - success

 - name: failure
   image: hub.evbj.easou.com/dev/drone-wechat:20200622
   settings:
     corpid: ww419ee4063735e1c0
     corp_secret: zpiRBLETH9eLwIMQ4eJ_r_dcm3BPSGeLHvTcft8Ot-M
     agent_id: 1000004
     title: "Pipeline ${DRONE_REPO_NAME} Failure"
     description: "${DRONE_BUILD_LINK} 部署失败，请检查配置！"
     msg_url: ${DRONE_BUILD_LINK}
     btn_txt: "否"
   when:
     status:
     - failure
```



```dockerfile
# Dockerfile
FROM mariadb

ENV MYSQL_DATABASE=nextcloud
ENV MYSQL_USER=nextcloud
```



```yaml
# deploy-tmp.yaml
# 不需要动
---
apiVersion: apps/v1
kind: $JNLP_CONTROL
metadata:
  name: $DRONE_REPO_NAME
  namespace: $JNLP_ENV
  labels:
    app: $DRONE_REPO_NAME
spec:
  serviceName: $DRONE_REPO_NAME
  replicas: $JNLP_REPLICAS
  selector:
    matchLabels:
      app: $DRONE_REPO_NAME
  template:
    metadata:
      labels:
        app: $DRONE_REPO_NAME
    spec:
      terminationGracePeriodSeconds: 180
      initContainers:
        - name: init
          image: $JNLP_REPO/dev/busybox
          command: ["chmod","777","-R","$JNLP_MOUNT_PATH"]
          imagePullPolicy: Always
          volumeMounts:
          - name: volume
            mountPath: $JNLP_MOUNT_PATH
      containers:
      - name: $DRONE_REPO_NAME
        image: $JNLP_REPO/$JNLP_ENV/$DRONE_REPO_NAME:$JNLP_TAG
        imagePullPolicy: Always
        ports:
        - containerPort: $JNLP_CONTAINER_PORT
          name: port
        volumeMounts:
        - name: volume
          mountPath: $JNLP_MOUNT_PATH
        livenessProbe:
          tcpSocket:
            port: $JNLP_CONTAINER_PORT
          initialDelaySeconds: $JNLP_LIVENESS_INIT
          periodSeconds: $JNLP_LIVENESS_PER
        readinessProbe:
          tcpSocket:
            port: $JNLP_CONTAINER_PORT
          initialDelaySeconds: $JNLP_READINESS_INIT
          periodSeconds: $JNLP_READINESS_PER
  volumeClaimTemplates:
  - metadata:
      name: volume
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: rbd
      resources:
        requests:
          storage: $JNLP_STORAGE_CAPACITY
---
apiVersion: v1
kind: Service
metadata:
  name: $DRONE_REPO_NAME-svc
  namespace: $JNLP_ENV
  labels:
    app: $DRONE_REPO_NAME-svc
spec:
  type: NodePort
  ports:
  - port: $JNLP_CONTAINER_PORT
    targetPort: $JNLP_CONTAINER_PORT
  selector:
    app: $DRONE_REPO_NAME
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: $DRONE_REPO_NAME-ingress
  namespace: $JNLP_ENV
  annotations:
    kubernetes.io/ingress.class: $JNLP_INGRESS
spec:
  rules:
  - host: $JNLP_ENV-$DRONE_REPO_NAME$JNLP_DOMAIN
    http:
      paths:
      - path: /
        backend:
          serviceName: $DRONE_REPO_NAME-svc
          servicePort: $JNLP_CONTAINER_PORT
```



2、进入 http://drone2.ieasou.cn/ 查看创建的项目并进行激活，激活后就可以通过提交项目来触发 webhook 进行构建了。

![image-20220113192000113](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220113192000113.png)



3、构建了一次，失败了，哈哈哈。明天来了在看吧。