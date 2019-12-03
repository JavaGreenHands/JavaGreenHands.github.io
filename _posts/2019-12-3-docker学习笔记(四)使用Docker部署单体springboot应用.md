---
date: 2019-12-3T17:29:33.242Z
layout: post
title: Docker学习笔记(四)使用Dokcer部署单体springboot应用
description: >-
  springboot整合docker
image: >-
  ../blogimg/docker/01/docker01.jpg
optimized_image: >-
  ../blogimg/docker/01/docker01.jpg
category: docker
tags:
  - docker
  - springboot
author: mrwhite
paginate: true
---

### 使用Docker部署单体springboot应用

总体步骤分为:

- 准备需要部署的springboot应用

  这是使用的springboot的应用非常简单,就只有一个简单的计数的接口,并且计数的值也并不会做存储,直接是在内存中存储的,基于springboot2.x的一个简单应用

  pom.xml

  ```xml
  <!-- parent依赖 -->
    <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.1.7.RELEASE</version>
          <relativePath/> <!-- lookup parent from repository -->
      </parent>
  
  <!--自定义镜像前缀 -->
    <properties>
          <java.version>1.8</java.version>
          <docker.image.prefix>docker.springboot</docker.image.prefix>
      </properties>
  <!-- jar包依赖 -->
  <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <optional>true</optional>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
      </dependencies>
  <!-- 构建配置 -->
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
                      <!--镜像名称 -->
                      <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                      <!-- 指定构建镜像的dockerfile-->
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

  应用主体接口

  ```java
  @RestController
  public class DemoController {
  
      @GetMapping("/get")
      public String count(){
  
          return "hello Docker! count is "+ Count.num.getAndAdd(1);
      }
  }
  
  ```

  Count 类

  ```java
  public class Count {
  
      public  static AtomicInteger num = new AtomicInteger(0);
  }
  ```

  Dockerfile编写

  ```dockerfile
  FROM openjdk:8-jdk-alpine
  
  VOLUME /tmp
  ADD docker-springboot.jar docker.jar
  ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/docker.jar"]
  ```

- 准备docker 环境,准备jdk和maven环境

  > 需要搭建Docker环境,jdk和maven环境,建议在linux上搭建环境,环境搭建是最基础的技能,不会的也可以自行安装虚拟机,问问度娘就可以

- 打包应用启动,确认应用无问题,确认环境安装无问题后,将应用上传至服务端,

  ```bash
  #登录服务器 安装上传下载插件
  yum -y install lrzsz
  # 在本地将应用打包,上传至服务器
  rz docker-springboot.tag.gz 
  ```

  解压文件,启动springboot应用

  ```bash
  tar -zxvf docker-springboot.tag.gz 
  
  cd docker-springboot
  # 打包应用
  mvn clean package
  # 启动应用
  java -jar docker-springboot.jar
  ```

  启动完成后直接在浏览器输入[http://192.168.89.129:8080/get](http://192.168.89.129:8080/get)

  返回

  ```
  hello Docker! count is 0
  ```

  每次访问该接口时count的值会递增

  证明项目无问题,如果出现接口访问不通的情况时,需要检查防火墙是否启动,可以选择关闭防火墙或者是在防火墙中加入8080端口的对外开放

- 使用docker的maven插件进行构建

在项目根目录执行命令,可以指定tag,也可以不指定tag

```bash
mvn clean package docker:build -DskipTest=true -DdockerImageTags=v1
```

出现以下日志代表构建成功

```bash
[INFO] Using authentication suppliers: [ConfigFileRegistryAuthSupplier]
[INFO] Copying /root/docker-springboot/target/docker-springboot.jar -> /root/docker-springboot/target/docker/docker-springboot.jar
[INFO] Copying src/main/docker/Dockerfile -> /root/docker-springboot/target/docker/Dockerfile
[INFO] Building image docker.springboot/docker-springboot
Step 1/4 : FROM openjdk:8-jdk-alpine
Trying to pull repository docker.io/library/openjdk ... 
Pulling from docker.io/library/openjdk
e7c96db7181b: Pull complete 
f910a506b6cb: Pull complete 
c2274a1a0e27: Pull complete 
Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
Status: Downloaded newer image for docker.io/openjdk:8-jdk-alpine
 ---> a3562aa0b991
Step 2/4 : VOLUME /tmp
 ---> Running in 24a544b029f7
 ---> e6ad57db6780
Removing intermediate container 24a544b029f7
Step 3/4 : ADD docker-springboot.jar docker.jar
 ---> e0963f46bf85
Removing intermediate container b7655ce29994
Step 4/4 : ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar /docker.jar
 ---> Running in 40c5b34021d6
 ---> 5a045353be28
Removing intermediate container 40c5b34021d6
Successfully built 5a045353be28
[INFO] Built docker.springboot/docker-springboot
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  03:36 min
[INFO] Finished at: 2019-08-21T15:41:25+08:00
[INFO] ------------------------------------------------------------------------
```

查看docker镜像

```bash
docker images

REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
docker.springboot/docker-springboot   v1              54fe385a0aca        2 minutes ago       123 MB

```

docker镜像构建成功!!!!

- 运行docker容器

  ```bash
  docker run -d -p 8080:8080  -t 54fe385a0aca
  ```

  查看dokcer 运行的容器

  ```bash
  docker ps 
  
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
  9531dfd453dd        54fe385a0aca        "java -Djava.secur..."   48 seconds ago      Up 47 seconds       0.0.0.0:8080->8080/tcp   epic_wiles
  ```

  再次从浏览器中访问接口 ,访问成功,使用docker部署springboot已经成功了

  ### 注意事项

  1. 注意Dockerfile的位置与pom.xml的位置一样

  2. 虚拟机的防火墙的设置,可能会导致应用启动并正常运行,但是接口却访问不通

  3. 防火墙的禁用或修改后,建议重启docker.          systemctl restart docker

     

  