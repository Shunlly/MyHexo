---
title: docker学习
date: 2021-10-17 15:31:01
tags: docker
categories: 学习
keywords: docker
description: docker的学习笔记
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# Docker学习笔记

## 1.Docker简介

Docker是一种运行于Linux和Windows上的软件，用于穿件、管理和编排容器。

Docker在Github上开发的Moby开源项目的一部分。

![image.png](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/1635746950373-4639ad99-25d4-446f-8ba2-64d1b4caa425.png)

dockerClient客户端

Docker Daemon守护进程

Docker Image镜像

DockerContainer容器

![image.png](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/1635747181750-953109a2-9823-4591-b8c7-a713fb3e1eeb.png)

## 2.Docker的优势

-   轻量，秒级的快速启动速度

-   简单，易用，活跃的社区&nbsp;

-   标准统一的打包/部署/运行方案

-   镜像支持增量分发，易于部署&nbsp;

-   易于构建，良好的REST API，也很适合自动化测试和持续集成

-   性能，尤其是内存和IO的开销

## 3.三要素

-   镜像（image）

镜像/容器

镜像就是模板，容器就是镜像的一个实例，例如：

```java
Person p1 = new Person()  
Person p2 = new Person()  
Person p3 = new Person()  
//Person都来自同一个对象，p1、p2、p3就是3个不同的实例  
//就是一个容器，即镜像就是一个类，容器就是一个一个对象
```



-   容器（container）

Docker利用容器独立运行的一个或一组应用。容器是用镜像创建的运行实例，可以把容器看成是一个简易版的Linux环境

-   仓库（respository）

仓库（Respository）是集中存放镜像文件的场所，仓库和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个 镜像，每个镜像有不同的标签（tag）。

仓库分为公开仓库（public）和私有仓库（Private）两种形式。最大的公开仓库是Docker Hub（https://hub.docker.com/）

## 4.Docker安装

-   查看自己的服务器上的系统版本

uname -r

cat /etc/redhat-release

-   根据服务器的系统不同，版本不同，安装过程会有所区别

[**](https://docs.docker.com/engine/install/centos/#install-using-the-repository)https://docs.docker.com/engine/install/centos/\#install-using-the-repository

这是官方文档的安装步骤。

```sh
# 卸载旧版本  
sudo yum remove docker \
docker-client \  
docker-client-latest \  
docker-common \  
docker-latest \  
docker-latest-logrotate \  
docker-logrotate \  
docker-engine  
#设置存储库  
#安装yum-utils包（提供yum-config-manager 实用程序）  
#并设置稳定存储库  
sudo yum install -y yum-utils  
sudo yum-config-manager \  
--add-repo \  
https://download.docker.com/linux/centos/docker-ce.repo  
#可选：启用夜间或测试存储库  
#开启  
sudo yum-config-manager --enable docker-ce-nightly  
#要启用测试通道，请运行以下命令：  
sudo yum-config-manager --enable docker-ce-test  
#关闭  
sudo yum-config-manager --disable docker-ce-nightly  
#安装docker引擎  
sudo yum install docker-ce docker-ce-cli containerd.io
```

-   可以配置阿里云的镜像仓库。具体的可以参考[*https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors*](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

## 5.Docker命令

-   帮助命令

    -   docker version

    -   docker info&nbsp;

    -   docker --help&nbsp;

-   镜像命令

    -   docker images 查看本地的镜像

        -   docker images -a 列出本地所有的镜像

        -   docker images -q 只显示镜像的摘要信息&nbsp;

        -   docker images --digests 显示镜像的摘要信息

        -   docker images --no-trunc 显示完整的镜像信息&nbsp;

    -   docker search xxx 搜索相应的镜像(xxx为镜像名字)

        -   docker search xxx -f stars=30 搜索星星数大于30的镜像

        -   docker search xxx -f stars=30 --no-trunc 显示详细信息&nbsp;

    -   docker pull xxx 下载镜像

    -   docker rmi xxx 删除镜像

        -   docker rmi -f xxx 强制删除

        -   docker rmi -f xxx\[:tag\] xxx\[:tag\] 多个删除&nbsp;

        -   docker rmi -f \$(docker images -q) 全部删除&nbsp;

-   容器命令（有镜像才能创建容器）

    -   docker pull centos 下载centos镜像

    -   docker run \[options\] xxx 新建并运行xxx镜像

        -   options: --name（容器新名字，为容器指定一个名称）

        -   options: -d（后台运行容器，并返回容器ID，也即启动守护式容器）&nbsp;

        -   options: -i（以交互模式运行容器，通常与-t同时使用）

        -   options: -t（为容器重新分配一个伪输入终端，通常与-i同时使用）&nbsp;

        -   options: -P（随机映射端口）

        -   options: -p（指定端口映射，有以下四种格式）

            -   ip:hostPort:containerPort

            -   ip::containerPort&nbsp;

            -   hostPort:containerPort

            -   containerPort&nbsp;

    -   docker ps \[options\] 查看正在运行的镜像

        -   options: -a（列出当前所有正在运行的容器+历史上运行过的）

        -   options: -l（显示最近创建的容器）&nbsp;

        -   options: -n（显示最近n个创建的容器）

        -   options: -q（静默模式，只显示容器编号）&nbsp;

        -   options: --no-trunc（不截断输出）&nbsp;

    -   exit 退出并关闭容器

    -   ctrl+p+q 退出但不关闭容器(windows)&nbsp;

    -   docker start 容器id 启动容器

    -   docker restart 容器id 重启容器&nbsp;

    -   docker stop 容器id 停止容器

    -   docker kill 容器id 强制停止&nbsp;

    -   docker rm 容器id 删除容器

        -   docker rm -f 容器id 强制删除

        -   docker rm -f \$(docker ps -a -q) 多个删除&nbsp;

        -   docker ps -a -q \| xargs docker rm 多个删除&nbsp;

    -   docker run -d centos 以后台进程的模式启动容器，这样做，导致容器前台没有运行应用，这样启动后，会立即自杀，因为容器觉得没有事情可做。解决方式：将要运行的程序以前台进程的形式运行。

    -   docker logs -f -t --tail 容器id 查看容器日志

        -   -t 是加入时间戳

        -   -f 跟随最新的日志大新&nbsp;

        -   --tail 数字，显示最后多少条&nbsp;

    -   docker top 容器id 查看容器内运行的进程

    -   docker inspect 容器id 查看容器内部细节（json字符串的形式）&nbsp;

    -   docker exec -it 容器id bashShell

        -   docker exec -it 容器id /bin/bash 直接进入目录（默认为/bin/bash）&nbsp;

    -   dcoker attach 容器id 重新进入容器

    -   docker cp 容器id: /tmp/yum.log /root 拷贝文件到root目录下

## 6.Docker镜像

-   docker 的镜像实际上是由一层一层的文件系统组成，这种层级的文件系统UnionFS。

-   ![image.png](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/1636004250376-b17d34f7-0313-4c04-b98e-3abee09a4772.png)

-   docker Commit的操作

## Docker容器数据卷

-   将运用与运行的环境打包形成容器运行，运行可以伴随着容器，但是我们可以对数据的要求希望是持久化的

-   容器之间是希望有可能共享数据&nbsp;

-   Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了。

-   为了能保存数据在docker中我们使用卷&nbsp;

-   命令添加容器卷

    -   docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名

    -   docker run -it -v /宿主机绝对路径目录:/容器内目录 ro 镜像名

## DockerFile解析

-   DockerFile是用来构建Docker镜像的构建文件，有由一系列命令和参数构成的脚本

-   构建三步骤：

    -   编写Dockerfile文件

    -   dockerfile build&nbsp;

    -   docker run&nbsp;

-   Dcokerfile内容基础知识

    -   1.每条保留字指令都必须为大写字母且后面要跟随至少一个参数

    -   2.指令按照从上到下，顺序执行&nbsp;

    -   3.\#表示注释

    -   4.每条指令都会创建一个新的镜像层。并对镜像进行提交&nbsp;

-   Docker执行Dockerfile的大致流程

    -   （1）docker从基础镜像运行一个容器

    -   （2）执行一条指令并对容器作出更改&nbsp;

    -   （3）执行类似docker commit的操作提交一个新的镜像层

    -   （4）docker再基于刚提交的镜像运行一个新容器&nbsp;

    -   （5）执行dockerfile中的下一条指令直到所有指令都执行完成&nbsp;

-   Dockerfile体系结构（保留字指令）

    -   FROM\|基础镜像，当前镜像是基于哪个镜像的&nbsp;

    -   MAINTAINER \|镜像维护者的姓名和邮箱地址

    -   RUN 容器构建时需要运行的命令&nbsp;

    -   EXPOSE 当前容器对外暴露出的端口

    -   WORKDIR 指定在创建容器之后，终端默认登陆的进来工作目录，一个落脚点&nbsp;

    -   ENV 用来在构建镜像过程中设置环境变量

    -   ADD 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包&nbsp;

    -   COPY 类似ADD，拷贝文件和目录到镜像中。

    -   VOLUME 容器数据卷，用于数据保存和持久化工作&nbsp;

    -   CMD 指定容器启动时要运行的命令，最后一个命令会顶替之前的

    -   ENTRYPOINT 指定容器启动时要运行的命令，在之前的命令执行后追加&nbsp;

    -   ONBUILD 当构建一个被继承的DockerFile时运行命令，父镜像子在被子继承后父镜像的onbuild被触发&nbsp;

-   案例

```dockerfile
FROM cetos  
MAINTAINER csh&lt;330289953@qq&gt;com&gt;  

ENV MYPATH /usr/local  
WORKDIR \$MYPATH  

RUN yum -y install vim  
RUN yum -y install net-tools  

EXPOSE 80  

CMD echo \$MYPATH  
CMD echo "success------------ok"  
CMD /bin/bash
```



## MySql安装

docker pull mysql:5.7下载5.7版本的mysql

运行：

docker run -p 3306:3306 --name mysql57 -v /opt/mysql/conf:/etc/mysql/conf.d -v /opt/mysql/logs:/logs -v /opt/mysql/data:/var/lib/mysql -e MYSQL\_ROOT\_PASSWORD=123456 -d mysql:5.7

## Redis安装

docker pull redis 将redis从docker上拉下来

运行：

docker run -p 6379:6379 -v /opt/myredis/data:/data -v /opt/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis redis-server /usr/local/etc/redis/redis.conf --appendonly yes

## 案例（DockerFile）

```
FROM centos  
MAINTAINER csh&lt;a330289953@gmail.com&gt;  
# 把宿主机当前上下文的c.text拷贝到容器/usr/local路径下  
COPY c.txt /usr/local/cincontainer.txt  
# 把java与tomcat添加到容器中  
ADD jdk-8u171-linux-x64.tar.gz /usr/local  
ADD apache-tomcat-9.0.8.tar.gz /usr.local  
# 安装vim编辑器  
RUN yum -y install vim  
# 设置工作访问时候的WORKDIR路径，登录落脚点  
ENV MYPATH /usr/local  
WORKDIR $MYPATH  
# 配置java与tomcat环境变量  
ENV JAVA_HOME /usr/local/jdk1.8.0_171  
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar  
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8  
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8  
ENV PATH $PAHT:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin  
# 容器运行时监听的端口  
EXPOSE 8080  
# 启动时运行tomcat  
# ENTRYPOIN ["usr/local/apache-tomcat-9.0.8/bin/startup.sh"]  
# CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.hs","run"]  
CMD /usr/lcoal/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/logs/catailna.out
```

```sh
docker run \\

-d \

-p 6379:6379 \\

--restart unless-stopped \\

-v /home/memect/project/tools/mydata/redis/data:/data \\

-v /home/memect/project/tools/mydata/redis/conf/redis.conf:/etc/redis/redis.conf \\

redis-server /etc/redis/redis.conf \\

redis

```

```sh
docker run -p 3306:3306 --name mysql8 -v /home/chenshuhang/tools/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /home/chenshuhang/tools/mysql/conf:/etc/mysql/conf.d -v /home/chenshuhang/tools/mysql/logs:/logs -v /home/chenshuhang/tools/mysql/data:/var/lib/mysql -e MYSQL\_ROOT\_PASSWORD=123456 -d mysql:8.0

docker run --name mysql -p 3306:3306 -e MYSQL\_ROOT\_PASSWORD=123456 -d mysql:8.0

ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

docker run -p 8848:8848 -v /home/memect/project/login\_util:/code login\_util

sudo docker run -p 6379:6379 --name redis -v /home/memect/project/tools/myredis/conf/redis.conf:/etc/redis/redis.conf -v /home/memect/project/tools/myredis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

设置密码

config set requirepass 123456

```
docker run -d \\

--name=kibana \\

--restart=always \\

-p 5601:5601 \\

-v /opt/tools/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \\

kibana:7.16.2
```

```
docker run -p 6379:6379 --name redis -v /home/memect/tools/redis/redis.conf:/etc/redis/redis.conf -v /home/memect/tools/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

```
docker run -p 3308:3306 --name mysql57 -v /Users/mac/canal/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /Users/mac/canal/mysql/logs:/logs -v /Users/mac/canal/mysql/data:/var/lib/mysql -e MYSQL\_ROOT\_PASSWORD=123456 -d mysql:5.7
```

