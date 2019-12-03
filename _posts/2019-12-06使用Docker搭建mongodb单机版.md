---
date: 2019-12-06 18:00:00
layout: post
title: Docker搭建mongoDB
description: >-
    docker搭建mongodb  
image: >-
  ../blogimg/docker/01/docker01.jpg
optimized_image: >-
  ../blogimg/docker/01/docker01.jpg
category: docker
tags:
  - docker-compose
  - mongodb
author: mrwhite
paginate: true
---

### 使用Docker搭建mongodb简单版

文件结构

```bash
├── docker-compose.yml

├── Dockerfile

└── setup

    └── setup.js

```

**setup.js**

用于初始化MongoDB

```javascript
db = db.getSiblingDB('gis');  // 创建一个名为"gis"的DB
db.createUser(  // 创建一个名为"shon"的用户，设置密码和权限
    {
        user: "guide",
        pwd: "guide",
        roles: [
            { role: "dbOwner", db: "gis"}
        ]
    }
);
db.createCollection("gis");  // 在"gis"中创建一个名为"gis"的Collection

```

**docker-compsoe文件**

```yaml
version: '3.1'  # 与镜像有关，这里只支持3.1
services:
   mongo:  # 会自动从Docker Hub上自动获取mongo这个镜像
    build: ./
    restart: always
    ports:
      - 9091:27017  # 本地端口(可自定义):容器内默认端口(mongo设定为27017)
    volumes:
      - ./setup:/docker-entrypoint-initdb.d/  # 本地文件路径:容器内映射路径
    environment:  # admin账号和密码
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
  # 如果不需要MongoDB的网页端，以下内容可以不加
   mongo-express:  # 会自动从Docker Hub上自动获取mongo-express这个镜像
    image: mongo-express
    restart: always
    ports:
      - 9092:8081  # 本地端口(可自定义):容器内默认端口(mongo-express设定为8080)
    environment:  # 这里只能使用与上方MONGO_INITDB_ROOT_USERNAME相同的root账号
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: admin
```

**Dockerfile**

用于初始化Docker容器

```dockerfile
FROM mongo
# 将本地的setup.js映射到Docker容器中
COPY ./setup/setup.js /docker-entrypoint-initdb.d/
```

**检查配置并启动**

```bash
# docker-compose 配置检查
docker-compose config
# 拉取镜像 
docker-compose pull 
# 后台启动 
docker-compose up -d
```

**验证**

浏览器直接输入安装mongo的ip+port即可 localhost:9092 可访问mongo的web端

```bash

# 查看正在运行的容器Id
sudo docker ps 

# 进入到docker的容器服务端
sudo docker exec -it b45b038cf854 bash

#链接mongo客户端
mongo -u guide -p guide --host localhost --authenticationDatabase gis

# 显示如下
root@b45b038cf854:/# mongo -u guide -p guide --host localhost --authenticationDatabase gis
MongoDB shell version v4.0.10
connecting to: mongodb://localhost:27017/?authSource=gis&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("004b943e-aa33-4510-a57a-59a838cc4998") }
MongoDB server version: 4.0.10
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user

```

>这种的mongodb仅仅适用于本机访问,如何开启远程连接,暂时没有搞明白原理,也可能是我的机器有问题,配置文件无论如何修改都不能够开启远程访问,如果哪位大神知道麻烦指点一下

[参考文章](https://blog.csdn.net/u011104991/article/details/81735960)