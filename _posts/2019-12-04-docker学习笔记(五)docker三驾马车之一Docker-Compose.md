---
date: 2019-12-04 17:29:33
layout: post
title: Docker学习笔记(五)Docker-compose的使用
description: >-
  docker-compose
image: >-
  ../blogimg/docker/01/docker01.jpg
optimized_image: >-
  ../blogimg/docker/01/docker01.jpg
category: docker
tags:
  - docker
  - spring-boot
  - docker-compose
author: mrwhite
paginate: true
---

### docker学习笔记(五)Docker-Compose

#### 简介

> Docker-compose是官方的开源项目,负责实现对Docker容器集群的快速编排

举个简单的例子,我们一个java Web应用,依赖的环境有jdk ,数据库使用的mysql,缓存数据库使用的redis,消息队列服务器使用的是rabbitmq,并且分为ABC三个应用协作运行,也就是说我们需要一台mysql服务器,一台rabbitmq服务器和一台redis服务器,Docker-Compose服务编排,也就是说我们使用Docker-compose自定义一个配置文件去配置这些环境,将环境安装在Docker中,并且编排好启动顺序,依赖,在以后应用服务移植时可以不用重复搭建环境,只需要安装Docker环境即可

Compose中有两个重要的概念:

- 服务(service):一个应用的容器,实际上可以包括若干运行相同镜像的容器实例,比如:ABC三个应用协作运行,需要三台mysql服务,可以从同一个镜像中运行三个不同的mysql实例即可

- 项目(project): 由一组关联应用容器组成的一个完整的业务单元,例如我们ABC三个应用协作,加上运行环境 redis实例,mq实例 ,mysql实例,这样构成一个项目的完整运行,是一组完整的业务单元

  

  > compose的默认管理对象是项目,也就是说通过命令对项目中的一组容器进行便捷的生命周期管理,compose项目由Python编写,实际上调用了Docker服务提供的API来对容器进行管理,因此,只要所有操作的平台支持DockerAPI,就可以利用Compose来进行编排管理

  **安装命令**

  ```bash
  #下载
  sudo curl -L https://github.com/docker/compose/releases/download/1.20.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  #安装
  chmod +x /usr/local/bin/docker-compose
  #查看版本
  docker-compose version
  # 显示
  mrwhite@mrwhite-PC:~$ docker-compose version
  docker-compose version 1.20.0, build ca8d3c6
  docker-py version: 3.1.3
  CPython version: 3.6.4
  OpenSSL version: OpenSSL 1.0.1t  3 May 2016
  
  ```

  **bash补全命令**

  ```bash
  curl -L https://raw.githubusercontent.com/docker/compose/1.8.0/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
  ```

  **卸载**

  如果是二进制包方式安装的,删除二进制文件即可

```bash
sudo rm /usr/local/bin/docker-compose
```

### 使用Docker-Compose部署springboot应用

现在有一个springboot应用,使用的依赖环境有 redis,我们需要的环境就是redis,java环境,使用docker-compose将java和redis的环境打包为一组服务部署.

**controller**

```java
/**
 * Docker-Compose应用测试类
 *
 * @author baijie
 * @date 2019-08-23
 */
@RestController
public class ComposeController {

    @Autowired
    private RedisTemplate<String,String> redisTemplate;

    String KEY =  "INCR_KEY" ;


    @GetMapping("/get/incr")
    public String getIncr(){
        String s = redisTemplate.opsForValue().get(KEY);
        if(StringUtils.isEmpty(s)){
            redisTemplate.opsForValue().set(KEY,"1");
            return "1";
        }else {
            Long increment = redisTemplate.opsForValue().increment(KEY);
            return increment.toString();
        }

    }
}
```

**application.yml**

```yaml
server:
  port: 9070
spring:
  redis:
    host: redis_cache
    port: 6379
    timeout: 60s
    database: 0
    lettuce:
      pool:
        max-active: 200
        max-idle: 20
        min-idle: 5
        max-wait: -1
```

**pom redis 依赖**

```xml
 <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>redis.clients</groupId>
                    <artifactId>jedis</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

**maven Docker 插件**

```xml
  <build>
        <finalName>docker-springboot</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- Docker maven plugin -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <imageTags>
                        <imageTag>v1</imageTag>
                    </imageTags>
                    <dockerDirectory>src/main/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
            <!-- Docker maven plugin -->
        </plugins>
    </build>
```

**springboot Dockerfile配置**

```dockerfile
FROM maven:3.5-jdk-8
```

**准备redis.conf的配置**

redis.conf

```
appendonly yes
daemonize no
```



创建文件夹并启动 

```bash
cd ~
mkdir docker-compose
cd docker-compose
```

**编写Docker-compose文件**

```yaml
version: "3"

services:
        app:
                build: ./app
                ports: 
                        - 9070:9018
                environment:
                        - TZ=Asia/Shanghai
                depends_on:
                        - redis_cache
                working_dir: /app
                volumes: 
                        - /etc/localtime:/etc/localtime:ro
                        - ./app:/app
                links:
                        - redis_cache
                command: mvn clean spring-boot:run
                
        redis_cache:
                restart: always
                ports:
                        - 6379:6379
                image: redis 
                container_name: redis_cache
                command: redis-server /usr/local/etc/redis/redis.conf
                volumes:
                        - ./redis_data/:/data/
                        - ./redis.conf:/usr/local/etc/redis/redis.conf
```

**更改项目目录结构**

将项目拷贝至docker-compose文件夹并命名为app,在其中创建一个Dockerfile

```yml
FROM maven:3.5-jdk-8
```

**启动Docker-compose**

```bash
# 拉取镜像
docker-compose pull 
```

启动项目

```bash
docker-compose up -d
```

**Docker-compose配置解释**

​	version: 

> "3" 代表使用第三代语法构建docker-compose编排服务文件

​	services:

> 代表一组服务，该标签下每一个服务是在运行时是一个容器

​	app:

> 服务名，根据我们自己的服务自定义

​	image :

> 指定我们的容器运行时从哪个镜像启动实例，					

​	build

>  代表构建的镜像的Dockerfile所在的目录,build和image在docker-compose文件中的同一个服务中必须定义一个

​	ports:

> 端口号配置	9070代表宿主机的端口，9018代表容器内部的端口，容器内部的端口与springboot内部配置的端口需要对应

​	environment:

> 容器运行时的环境配置，"SPRING_PROFILES_ACTIVE=dev"  代表启动springboot应用时指定的配置文件,- TZ=Asia/Shanghai配置容器的时区，防止容器内时间与宿主机时间不同步

​	working_dir：

> 代表容器内应用的工作目录

​	volumes：

> 容器内部的目录挂载，例如:	- ./app  ./app代表宿主机目录，/app代表容器内的目录

 	depends_on: 

>服务依赖,在服务下此标签的意思是依赖于以下服务

​	restart: 

>always 代表如果服务启动不成功会一直尝试

​	container_name:

>指定容器的名称,如果没有这个配置,compose会默认分配一个容器名称

​	command:

>表示以这个命令启动这个服务

​	links:

>容器的链接,docker-compose中的每一个服务就是一个容器,而容器的环境是隔离的,容器相互依赖就需要通信,links是为了保持两个容器之间的通信,而且在容器内部通过compose配置links可以在容器内部使用服务名+端口来访问其他容器

**Docker-compose 命令详解**

```bash
#查看帮助
docker-compose -h

# -f  指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。
docker-compose -f docker-compose.yml up -d 

#启动所有容器，-d 将会在后台启动并运行所有的容器
docker-compose up -d

#停用移除所有容器以及网络相关
docker-compose down

#查看服务容器的输出
docker-compose logs

#列出项目中目前的所有容器
docker-compose ps

#构建（重新构建）项目中的服务容器。服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db。可以随时在项目目录下运行 docker-compose build 来重新构建服务
docker-compose build

#拉取服务依赖的镜像
docker-compose pull

#重启项目中的服务
docker-compose restart

#删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器。
docker-compose rm 

#在指定服务上执行一个命令。
docker-compose run ubuntu ping docker.com

#设置指定服务运行的容器个数。通过 service=num 的参数来设置数量
docker-compose scale web=3 db=2

#启动已经存在的服务容器。
docker-compose start

#停止已经处于运行状态的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器。
docker-compose stop
```

### 参考

[Docker Compose + Spring Boot + Nginx + Mysql 实践](http://www.ityouknow.com/springboot/2018/03/28/dockercompose-springboot-mysql-nginx.html)















