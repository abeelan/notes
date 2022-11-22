本文通过 docker 在服务器上搭建 sonic 真机平台 simple 版本。

部署文档：https://sonic-cloud.gitee.io/#/Deploy



### 1、创建数据库

```bash
$ docker run --name=mysql -it -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e MYSQL_ROOT_HOST=% -d mysql:5.7 --character-set-server=utf8 --collation-server=utf8_general_ci 

# 进入mysql容器，创建数据库
$ docker exec -it mysql /bin/bash
$ mysql -u root -p
mysql> create database sonic; 

# 修改字符集和字符排序规则(没用)
# mysql> SELECT CONCAT('ALTER TABLE ', table_name, ' CONVERT TO CHARACTER SET  utf8 COLLATE utf8_general_ci;')
# FROM information_schema.TABLES
# WHERE TABLE_SCHEMA = 'sonic';

# 通过 navicat 链接，创建数据库
# 字符集为utf8，排序规则为utf8_general_ci
```



### 2、按照提示修改 docker-compose 文件





