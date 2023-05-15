---
title: jenkins安装
date: 2022-10-14 14:21:11
tags: jenkins
categories: 安装
keywords: jenkins
description: jenkins安装
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# jenkins安装

| 序号 | 环境准备 | 版本        |
| ---- | -------- | ----------- |
| 1    | jdk      | Jdk11       |
| 2    | jenkins  | 2.346.1     |
| 3    | Ubantu   | 20.04桌面版 |



> 注意：jenkins安装时，jdk版本要与jenkins的版本匹配

jdk和jenkins对应的版本如下图所示：

![image-20221014102439062](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221014102439062.png)

在安装jenkins时的过程中得注意，在最新版本，也就是**2.361.1**这个版本中，jenkins的默认配置是jenkins用户（jenkins在安装的时候会自动创建一个名叫jenkins的用户），***这会导致在执行shell脚本命令，或者安装某些插件时，没有权限***，jenkins的配置文件所在地址/etc/default/jenkins，

- Jenkins2.361.1的版本（截止2022.10.14最新的版本）

  ```
  JENKINS_USER=$NAME
  JENKINS_GROUP=$NAME
  ```

- Jenkins2.346.1的版本

  ```
  JENKINS_USER=root
  JENKINS_GROUP=root
  ```

- 2.346.1具体的配置信息如下：

  ```
  # defaults for Jenkins automation server
  
  # pulled in from the init script; makes things easier.
  NAME=jenkins
  
  # arguments to pass to java
  
  # Allow graphs etc. to work even when an X server is present
  JAVA_ARGS="-Djava.awt.headless=true"
  
  #JAVA_ARGS="-Xmx256m"
  
  # make jenkins listen on IPv4 address
  #JAVA_ARGS="-Djava.net.preferIPv4Stack=true"
  
  PIDFILE=/var/run/$NAME/$NAME.pid
  
  # user and group to be invoked as (default to jenkins)
  JENKINS_USER=root
  JENKINS_GROUP=root
  
  # location of the jenkins war file
  JENKINS_WAR=/usr/share/java/$NAME.war
  
  # jenkins home location
  JENKINS_HOME=/var/lib/$NAME
  
  # set this to false if you don't want Jenkins to run by itself
  # in this set up, you are expected to provide a servlet container
  # to host jenkins.
  RUN_STANDALONE=true
  
  # log location.  this may be a syslog facility.priority
  JENKINS_LOG=/var/log/$NAME/$NAME.log
  #JENKINS_LOG=daemon.info
  
  # Whether to enable web access logging or not.
  # Set to "yes" to enable logging to /var/log/$NAME/access_log
  JENKINS_ENABLE_ACCESS_LOG="yes"
  
  # OS LIMITS SETUP
  #   comment this out to observe /etc/security/limits.conf
  #   this is on by default because http://github.com/jenkinsci/jenkins/commit/2fb288474e980d0e7ff9c4a3b768874835a3e92e
  #   reported that Ubuntu's PAM configuration doesn't include pam_limits.so, and as a result the # of file
  #   descriptors are forced to 1024 regardless of /etc/security/limits.conf
  MAXOPENFILES=8192
  
  # set the umask to control permission bits of files that Jenkins creates.
  #   027 makes files read-only for group and inaccessible for others, which some security sensitive users
  #   might consider benefitial, especially if Jenkins runs in a box that's used for multiple purposes.
  #   Beware that 027 permission would interfere with sudo scripts that run on the master (JENKINS-25065.)
  #
  #   Note also that the particularly sensitive part of $JENKINS_HOME (such as credentials) are always
  #   written without 'others' access. So the umask values only affect job configuration, build records,
  #   that sort of things.
  #
  #   If commented out, the value from the OS is inherited,  which is normally 022 (as of Ubuntu 12.04,
  #   by default umask comes from pam_umask(8) and /etc/login.defs
  
  # UMASK=027
  
  # port for HTTP connector (default 8080; disable with -1)
  HTTP_PORT=8080
  
  
  # servlet context, important if you want to use apache proxying
  PREFIX=/$NAME
  
  # arguments to pass to jenkins.
  # full list available from java -jar jenkins.war --help
  # --javaHome=$JAVA_HOME
  # --httpListenAddress=$HTTP_HOST (default 0.0.0.0)
  # --httpPort=$HTTP_PORT (default 8080; disable with -1)
  # --httpsPort=$HTTP_PORT
  # --argumentsRealm.passwd.$ADMIN_USER=[password]
  # --argumentsRealm.roles.$ADMIN_USER=admin
  # --webroot=~/.jenkins/war
  # --prefix=$PREFIX
  
  JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT"
  ```

安装命令：

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install fontconfig openjdk-11-jre
sudo apt-get install jenkins==2.346.1
```

启动jenkins

```
sudo systemctl start jenkins
```

查看jenkins状态

```
sudo systemctl status jenkins
```

关闭jenkins

```
sudo systemctl stop jenkins
```

重启jenkins

```
sudo systemctl restart jenkins
```

jenkins的访问网址，jenkins默认的端口是8080

http://127.0.0.1:8080

