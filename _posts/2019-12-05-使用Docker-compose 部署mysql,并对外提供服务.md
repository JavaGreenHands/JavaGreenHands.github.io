---
date: 2019-12-05 18:00:00
layout: post
title: 使用Docker-compose部署mysql
description: >-
  docker-compose中使用mysql
image: >-
  ../blogimg/docker/01/docker01.jpg
optimized_image: >-
  ../blogimg/docker/01/docker01.jpg
category: docker
tags:
  - docker-compose
  - mysql
author: mrwhite
paginate: true
---

### 使用Docker-compose 部署mysql,并对外提供服务

**1.准备环境**

- 安装 Docker 

- 安装 Docker-compose

**2.编写docker-compose文件**

- 创建docker-compose-mysql.yaml文件

  ```bash
  mkdir docker-mysql
  cd docker-mysql
  touch docker-compose-mysql.yaml
  ```

- 编写Docker-compose文件

  ```yaml
  version: "3"
  
  services:
          mysql:
                  image: mysql:5.7
                  container_name: mysql_1
                  ports:
                          - 3308:3306
                  environment:
                          MYSQL_DATABASE: demo_db
                          MYSQL_USER: demo_db
                          MYSQL_PASSWORD: demo_db
                          MYSQL_ROOT_PASSWORD: root
                  volumes: 
                          - ./data/databases_root/demo_db:/var/lib/mysql
                          - /etc/localtime:/etc/localtime:ro
                  restart: always 
                  command: [
                          '--character-set-server=utf8',
                          '--collation-server=utf8_general_ci',
                          '--sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
                          ]
  ```

**3.根据配置文件启动docker-mysql的镜像**

- 拉取mysql镜像

  ```bash
  # -f 指定配置文件
  sudo docker-compose -f docker-compose-mysql.yml  pull
  ```

- 启动

  ```bash
  sudo docker-compose -f docker-compose-mysql.yml  up -d
  ```

- 查看服务状态

  ```bash
  sudo docker ps
  
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
  3ba379f492bb        mysql:5.7           "docker-entrypoint..."   7 seconds ago       Up 6 seconds        33060/tcp, 0.0.0.0:3308->3306/tcp   mysql_1
  
  
  ```

- 访问

  ```bash
  # 进入到mysql容器中
  sudo docker exec -it 3ba379f492bb bash
  
  # 使用mysql客户端链接
  root@3ba379f492bb:/# mysql -u demo_db -p   
  Enter password: 
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 2
  Server version: 5.7.27 MySQL Community Server (GPL)
  
  Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
  
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  
  mysql> 
  mysql> 
  mysql> 
  ```

**4.修改mysql配置**

更改使用root用户登录mysql,赋予用户权限

```bash
# 授权允许远程访问mysql
GRANT ALL ON *.* TO demo_db@'%' IDENTIFIED BY 'demo_db' WITH GRANT OPTION; 
# 刷新权限
flush privileges;
```

**5.启动并测试**

```bash
# 登录
mysql -udemo_db -p -h127.0.0.1 -P3308
# 查看所有数据库
show databases;
# 显示
+--------------------+
| Database           |
+--------------------+
| information_schema |
| demo_db            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

>使用Docker 安装mysql特别方便,在安装mysql的过程中并没有什么难度,只是需要对Docker-compose的配置文件熟悉即可,注意的点就是数据文件的挂载和端口对外的暴露









