

yap-pro.sh

```shell
# https://hub.docker.com/r/yapipro/yapi

# 创建容器网络
docker network create yapi

# 删除历史构建容器
docker rm -f yapi
docker rm -f mongodb

# 启动 mongoDB
docker run -d \
  --name mongodb \
  --restart always \
  --net=yapi \
  -p 27017:27017 \
  -v $pwd/volumes/mongodb_bak:/data/db \
  mongo:4.4.4

# 启动 yapi
docker run -d \
  --name yapi \
  -p 3000:3000 \
  --link mongodb:mongo \
  --restart always \
  --net=yapi \
  -e YAPI_ADMIN_ACCOUNT=admin@easou.com \
  -e YAPI_ADMIN_PASSWORD=admin \
  -e YAPI_CLOSE_REGISTER=true \
  -e YAPI_LDAP_LOGIN_ENABLE=false \
  -e YAPI_MAIL_ENABLE=false \
  -e YAPI_DB_SERVERNAME=mongodb \
  -e YAPI_DB_PORT=27017 \
  -e YAPI_DB_DATABASE=yapi \
  yapipro/yapi \
  # server/app.js
```



config.json

```json
{
    "port":3000,
    "adminAccount":"admin@easou.cn",
    "timeout": 120000,
    "closeRegister":true,
    "db":{
        "servername":"mongo",
        "port":27017,
        "DATABASE":"yapi",
        "user": "yapi",
        "pass": "yapi123456",
        "authSource": ""
    },
    "mail":{
        "ena:ble":false
    },
    "ldapLogin":{
        "enable":false
    }
}
```

