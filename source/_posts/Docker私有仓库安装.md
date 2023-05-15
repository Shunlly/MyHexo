---
title: Docker私有仓库安装
date: 2022-10-14 13:21:01
tags: docker
categories: 安装
keywords: docker
description: Docker私用仓库安装
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# Docker私有仓库安装

> 注意：如果使用ubantu安装的docker，这个默认是通过snap安装的，它安装的权限只能控制当前用户的目录创建文件夹，例如：如果是root安装的docker，那么它只能在/root这个目录下创建文件夹，不能操纵/下的其他目录

- 环境准备

  | 序号 | 环境准备 | 版本         |
  | ---- | -------- | ------------ |
  | 1    | Ubantu   | 20.04_server |
  | 2    | Docker   | 最新的       |
  | 3    | registry | 最新的       |

- docker安装

  卸载旧版本

  ```
  apt-get remove docker docker-engine docker.io containerd runc
  ```

  安装前提依赖

  ```
  apt update
  apt-get install ca-certificates curl gnupg lsb-release
  ```

  安装GPG证书

  ```
  curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
  ```

  写入软件源信息

  ```
  add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
  ```

  安装最新版本

  ```
  apt-get install docker-ce docker-ce-cli containerd.io
  ```

  配置用户组

  ```
  groupadd docker
  ```

  启动docker

  ```
  systemctl start docker
  ```

  docker换源

  ```
  # 修改 /etc/docker/daemon.json (如果该文件不存在，则创建)
  {
      "registry-mirrors": [
          "https://hub-mirror.c.163.com"
  	]
  }
  ```

- 私有仓库搭建

  查看registry版本

  ```
  docker search registry
  ```

  安装registry

  ```
  docker pull registry
  ```

  创建本地目录，用于映射本地目录进docker里，目的是，当容器崩溃时，数据还在，随时以启用一个新容器替换

  ```
  mkdir -p /data/dockerhub
  ```

  启动registry

  ```
  docker run -d -v /data/dockerhub:/var/lib/registry -p 5000:5000 --restart=always --name dockerhub-registry registry
  ```

  访问网址http://ip:5000/v2，如果出现以下页面说明正常

  ```
  [root@dockerhub250 ~]# curl http://ip:5000/v2/
  {}
  ```

  server端daemon.json文件配置

  ```json
  {
  
  "insecure-registries":["sz-memect.tpddns.cn:5000"]
  
  }
  ```

  client端daemon.json文件配置

  ```
  {
  
  "insecure-registries":["sz-memect.tpddns.cn:5000"]
  
  }
  ```

  上传镜像至私有仓库

  ```
  [root@dockerhub250 ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  nginx               latest              4bb46517cac3        8 days ago          133MB
  registry            latest              2d4f4b5309b1        2 months ago        26.2MB
  #将要推送至私有仓库的docker镜像做标识
  [root@dockerhub250 ~]# docker tag nginx:latest 172.16.1.250:5000/nginx:latest
  [root@dockerhub250 ~]# docker images
  REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
  nginx                     latest              4bb46517cac3        8 days ago          133MB
  172.16.1.250:5000/nginx   latest              4bb46517cac3        8 days ago          133MB
  registry                  latest              2d4f4b5309b1        2 months ago        26.2MB
  #通过 docker push 命令将 nginx 镜像 push到私有仓库
  [root@dockerhub250 ~]# docker push 172.16.1.250:5000/nginx:latest
  The push refers to repository [172.16.1.250:5000/nginx]
  550333325e31: Pushed
  22ea89b1a816: Pushed
  a4d893caa5c9: Pushed
  0338db614b95: Pushed
  d0f104dc0a1f: Pushed
  latest: digest: sha256:179412c42fe3336e7cdc253ad4a2e03d32f50e3037a860cf5edbeb1aaddb915c size: 1362
  #查看是否上传成功
  [root@dockerhub250 ~]# curl http://127.0.0.1:5000/v2/_catalog
  {"repositories":["nginx"]}
  #查看镜像信息
  [root@dockerhub250 ~]# curl http://172.16.1.250:5000/v2/nginx/tags/list
  {"name":"nginx","tags":["latest"]}
  ```

  从其他内网机器验证拉取镜像

  ```
  [root@k8snode172 ~]# docker pull 172.16.1.250:5000/nginx
  Using default tag: latest
  latest: Pulling from nginx
  bf5952930446: Pull complete
  cb9a6de05e5a: Pull complete
  9513ea0afb93: Pull complete
  b49ea07d2e93: Pull complete
  a5e4a503d449: Pull complete
  Digest: sha256:179412c42fe3336e7cdc253ad4a2e03d32f50e3037a860cf5edbeb1aaddb915c
  Status: Downloaded newer image for 172.16.1.250:5000/nginx:latest
  172.16.1.250:5000/nginx:latest
  [root@k8snode172 ~]# docker images
  REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
  172.16.1.250:5000/nginx                     latest              4bb46517cac3        8 days ago          133MB
  ```

  

