---
date: 2017-10-17T17:29:33.242Z20
layout: post
title: Docker学习笔记(三)Dockerfile的概念及其编写
description: >-
  Dockerfile的基本概念
image: >-
  ../blogimg/docker/01/docker01.jpg
optimized_image: >-
  ../blogimg/docker/01/docker01.jpg
category: docker
tags:
  - dockerfile
author: mrwhite
paginate: true
---



### Dockerfile的概念及其编写

 Dockerfile可以允许用户创建自定义的镜像

**1基本结构**

------

Dockerfile由一行行命令组成，并且支持以#开头的注释行，一般，Dockerfile分为4部分：

- 基础镜像信息

- 维护者信息

- 镜像操作指令

- 容器启动执行指令

  **指令**

  ----------------

  **1. FROM**

  格式为 FROM <image> 或 FROM<image>:<tag>

  > 第一条指令必须为FROM指令，并且，如果同一个Dockerfile中创建多个镜像，可以使用多个FROM指令

  **2.MAINTAINER**

  格式为 MAINTAINER<name>,指定维护者的信息

  **3.RUN**

  格式为RUN<command> 或 RUN ["executable", "param1", "param2"] ,

  > 前者将在shell终端中运行命令，即/bin/sh  -c,后者使用exec执行。指定使用其他终端可以通过这种方式实现

  **4.CMD**

  支持三种格式:

  CMD ["executable","param1","param2"] 使用 exec 执行,推荐方式;

  CMD command param1 param2 在 /bin/sh 中执行,提供给需要交互的应用;

  CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数;

  > 指定启动容器时执行的命令,每个Dockerfile只能有一条CMD命令,如果指定了多条命令,只有最后一条会被执行,如果用户启动容器时指定了运行命令,则会覆盖掉CMD指定的命令

  **5.EXPOSE**

  格式为 EXPOSE <port> [<port>...]

  > 告诉Docker服务端容器暴露的端口号,供系统互联使用,在容器启动时需要通过-P,Docker主机会自动分配一个端口转发到指定的端口

  **6.ENV**

  格式为ENV<key> <value>.指定一个环境变量,会被后续RUN命令使用,并在容器运行时保持

  **7.ADD**

  格式为 ADD<src> <dest>

  > 该命令将复制指定的<src>到容器中的<dest> ,其中<src>可以是Dockerfile所在目录的一个相对路径,也可以是一个url,还可以是一个tar文件(自动解压为目录)

  **8.COPY**

  复制本地主机的 <src> (为 Dockerfile 所在目录的相对路径)到容器中的 <dest> 。
  当使用本地目录为源目录时,推荐使用 COPY 。

  **9.ENTRYPOINT**

  两种格式

  1. ENTRYPOINT ["executable", "param1", "param2"]
  2. ENTRYPOINT command param1 param2 (shell中执行)。

  > 配置容器启动后执行的命令,并且不可被 docker run 提供的参数覆盖。每个 Dockerfile 中只能有一个 ENTRYPOINT ,当指定多个时,只有最后一个起效

  **10.VOLUME**
  格式为 VOLUME ["/data"]

  > 创建一个可以从本地主机或其他容器挂载的挂载点,一般用来存放数据库和需要保持的数据等

  **11.USER**
  格式为 USER daemon

  > 指定运行容器时的用户名或 UID,后续的 RUN 也会使用指定用户。当服务不需要管理员权限时,可以通过该命令指定运行用户。并且可以在之前创建所需要的用户,例如: RUN groupadd -r postgres && useradd -r -g postgres postgres 。要临时获取管理员权限可以使用 gosu ,而不推荐 sudo 。

  **12.WORKDIR**

  格式为 WORKDIR /path/to/workdir

  > 为后续的 RUN 、 CMD 、 ENTRYPOINT 指令配置工作目录。可以使用多个 WORKDIR 指令,后续命令如果参数是相对路径,则会基于之前命令指定的路径。例如:

  ```
  WORKDIR /a
  WORKDIR b
  WORKDIR c
  RUN pwd
  ```

  > 则最终路径为 /a/b/c

  **13.ONBUILD**

  格式为 ONBUILD [INSTRUCTION]

  配置当所创建的镜像作为其它新创建镜像的基础镜像时,所执行的操作指令。

  配置当所创建的镜像作为其它新创建镜像的基础镜像时,所执行的操作指令。
  例如,Dockerfile 使用如下的内容创建了镜像 image-A 。

  ```
  [...]
  
  ONBUILD ADD . /app/src
  
  ONBUILD RUN /usr/local/bin/python-build --dir /app/src
  
  [...]
  
  ```

  如果基于 image-A 创建新的镜像时,新的Dockerfile中使用 FROM image-A 指定基础镜像时,会自动执行ONBUILD 指令内容,等价于在后面添加了两条指令。

  ```
  FROM image-A
  
  #Automatically run the following
  
  ADD . /app/src
  
  RUN /usr/local/bin/python-build --dir /app/src
  
  ```

  使用 ONBUILD 指令的镜像,推荐在标签中注明,例如 ruby:1.9-onbuild 。

  

编写完成 Dockerfile 之后,可以通过 docker build 命令来创建镜像。
基本的格式为 docker build [选项] 路径 ,该命令将读取指定路径下(包括子目录)的 Dockerfile,并将
该路径下所有内容发送给 Docker 服务端,由服务端来创建镜像。因此一般建议放置 Dockerfile 的目录为空
目录。也可以通过 .dockerignore 文件(每一行添加一条匹配模式)来让 Docker 忽略路径下的目录和文
件。
要指定镜像的标签信息,可以通过 -t 选项,例如

```
$ sudo docker build -t myrepo/myapp /tmp/test1/
```



### 参考
[Docker-从入门到实践](https://github.com/yeasy/docker_practice)