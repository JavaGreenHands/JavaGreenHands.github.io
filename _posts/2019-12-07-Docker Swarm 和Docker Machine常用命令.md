---
date: 2019-12-07 18:00:00
layout: post
title: DockerSwarm和DockerMachine常用命令
description: >-
   docker常用命令
image: >-
  ../blogimg/docker/01/docker01.jpg
optimized_image: >-
  ../blogimg/docker/01/docker01.jpg
category: docker
tags:
  - docker-compose
  - docker-machine
author: mrwhite
paginate: true
---

### Docker Swarm 和Docker Machine常用命令

**1.Docker swarm 常用命令**

初始化swarm集群,宿主机存在多个网卡时需要指定IP地址,执行docker swarm init 的节点自动成为管理节点

```bash
docker swarm init --advertise-addr [IP]
```

查看 swarm 集群状态

```bash
docker info
```

查看集群节点信息

```bash
docker node ls
```

添加工作节点

```bash
docker swarm join --token [创建工作节点时的token] [IP:prot]
```

```bash
# 管理配置文件
docker config
    # 查看已创建配置文件
    - docker config ls
    # 将已有配置文件添加到docker配置文件中
    - docker config create docker 配置文件名 本地配置文件
 # 管理swarm节点
docker node
    # 查看集群中的节点
    - docker node ls
    # 将manager角色降级为worker
    - docker node demote 主机名
    # 将worker角色升级为manager
    - docker node promote 主机名
    # 查看节点的详细信息，默认json格式
    - docker node inspect 主机名
          # 查看节点信息平铺格式
          - docker node inspect --pretty 主机名
    # 查看运行的一个或多个及节点任务数，默认当前节点
    - docker node ps
    # 从swarm中删除一个节点
    - docker node rm 主机名
    # 更新一个节点
    - docker node update
          # 对节点设置状态（“active”正常|“pause”暂停|“drain”排除自身work任务）
          - docker node update --availability
 # 管理敏感数据存储
docker secret
# 服务栈，栈的形式，一般作为编排使用，格式与docker compose相同。
docker stack
    # 通过.yml文件指令部署
    - docker stack deploy -c 文件名.yml 编排服务名
    # 查看编排服务
    - docker stack ls
 docker stack 不支持使用参数
# 作为集群的管理
docker swarm
    # 初始化一个swarm
    - docker swarm init
          # 指定初始化ip地址节点
          - docker swarm init --advertise-addr 管理端IP地址
          # 去除本地之外的所有管理器身份
          - docker swarm init --force-new-cluster
    # 将节点加入swarm集群，两种加入模式manager与worker
    - docker swarm join
          # 工作节点加入管理节点需要通过join-token认证
          - docker swarm join-token
          # 重新获取docker获取初始化命令
          - docker swarm join-token worker
    # 离开swarm
    - docker swarm leave
    # 对swarm集群更新配置
    - docker swarm update
# 服务管理
docker service
    # 创建一个服务
    - docker service create
          # 创建的副本数
          - docker service create --replicas 副本数
          # 指定容器名称
          - docker service create --name 名字
          # 每次容器与容器之间的更新时间间隔。
          - docker service create --update-delay s秒
          # 更新时同时并行更新数量，默认1
          - docker service create --update-parallelism 个数
          # 任务容器更新失败时的模式，（“pause”停止|”continue“继续），默认pause。
          - docker service create --update-failure-action 类型
          # 每次容器与容器之间的回滚时间间隔。
          - docker service create --rollback-monitor 20s
          # 回滚故障率如果小于百分比允许运行
          - docker service create --rollback-max-failure-ratio .数值（列“.2”为%20）
          # 添加网络
          - docker service create --network 网络名
          # 创建volume类型数据卷
          - docker service create --mount type=volume,src=volume名称,dst=容器目录
          # 创建bind读写目录挂载
          - docker service create --mount type=bind,src=宿主目录,dst=容器目录
          # 创建bind只读目录挂载
          - docker service create --mount type=bind,src=宿主目录,dst=容器目录,readonly
          # 创建dnsrr负载均衡模式
          - docker service create --endpoint-mode dnsrr 服务名
          # 创建docker配置文件到容器本地目录
          - docker service create --config source=docker配置文件,target=配置文件路径
          # 创建添加端口
          - docker service create --publish 暴露端口:容器端口 服务名
    # 查看服务详细信息，默认json格式
    - docker service inspect
          # 查看服务信息平铺形式
          - docker service inspect --pretty 服务名
    # 查看服务内输出
    - docker service logs
    # 列出服务
    - docker service ls
    # 列出服务任务信息
    - docker service ps　　　　
          # 查看服务启动信息
          - docker service ps 服务名
          # 过滤只运行的任务信息
          - docker service ps -f "desired-state=running" 服务名
    # 删除服务
    - docker service rm
    # 缩容扩容服务
    - docker service scale
          # 扩展服务容器副本数量
          - docker service scale 服务名=副本数
    # 更新服务相关配置
    - docker service update
          # 容器加入指令
          - docker service update --args “指令” 服务名
          # 更新服务容器版本
          - docker service update --image 更新版本 服务名         
         # 回滚服务容器版本
         - docker service update --rollback 回滚服务名
          # 添加容器网络
          - docker service update --network-add 网络名 服务名
          # 删除容器网络
          - docker service update --network-rm 网络名 服务名
          # 服务添加暴露端口
          - docker service update --publish-add 暴露端口:容器端口 服务名
          # 移除暴露端口
          - docker service update --publish-rm 暴露端口:容器端口 服务名
          # 修改负载均衡模式为dnsrr
          - docker service update --endpoint-mode dnsrr 服务名
```



**2.Docker-Machine常用命令**

```bash
#列出当前Docker虚拟主机
docker-machine  ls

#创建虚拟主机 --driver 指定驱动
docker-machine create --driver virtualbox dev

#显示连接到某个主机需要的环境变量 manager1为创建的虚拟主机名称
docker-machine env [manager1]

#显示虚拟主机的信息 manager1为创建的虚拟主机名称
docker-machine inspect [manager1]

#显示虚拟主机的IP manager1为创建的虚拟主机名称,可指定多个
docker-machine ip [manager1] [worker1]

#停止虚拟主机的运行
docker-machine kill [manager1]

#挂载虚拟主机目录
mkdir foo #创建一个文件夹
docker-machine ssh [manager1] [mkdir foo]
docker-machine mount [-u] [manager1:/home/docker/foo foo] #挂载本地foo目录到虚拟机中的foo中,-u代表卸载,不加就是挂载
touch foo/bar #宿主机创建一个文件
docker-machine ssh [manager1] [ls foo] # 查看是否挂载成功

#error1 "You must have a copy of the sshfs binary locally to use the mount feature"  需要安装sshfs 根据系统的不同自行安装
#error2 fuse: mountpoint is not empty
#fuse: if you are sure this is safe, use the 'nonempty' mount option
#exit status 1 挂点必须是空文件夹

# 重启虚拟主机
docker-machine resart [manager1]

# 删除虚拟主机
docker-machine rm [manager1]

# 从虚拟机上下载文件 
docker-machine scp manager1:/home/docker/foo/bar ./scpdir/

# 登录虚拟主机
docker-machine ssh [manager1]

# 启动虚拟主机
docker-machine start [manager1]

# 查看虚拟主机的状态
docker-machine status [manager1] 

# 停止虚拟主机
docker-machine stop [manager1]
```

### 参考

[Docker Swarm 常用命令](https://www.jianshu.com/p/979af789938a)

[Docker官方文档Docker-Machine命令](https://docs.docker.com/machine/reference/stop/)

[Docker官方文档Docker-swarm 命令](https://docs.docker.com/swarm/reference/create/)

