---
title: Harbor安装
date: 2022-10-14 14:11:01
tags: Harbor
categories: 安装
keywords: Harbor
description: Harbor安装
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# Harbor安装

| 序号 | 环境准备       | 版本         |
| ---- | -------------- | ------------ |
| 1    | Ubantu         | 20.04_server |
| 2    | docker         | 最新         |
| 3    | Docker-compose | 最新         |



- [harbor项目地址](https://github.com/goharbor/harbor)

  ![image-20221014113448665](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221014113448665.png)

下载harbor-online-install-v2.6.1.tgz，这个会在安装的时候从公网上下载一些必要的工具

如果使用的是harbor-offline-install-v2.6.1.tgz，那么建议提前将一些harbor集成的工具准备好，不然可能会安装失败

解压harbor-online-install-v2.6.1.tgz

```
tar -zxvf harbor-online-install-v2.6.1.tgz
```

harbor默认工作方式是http，但是这只能在页面访问，默认harbor推送拉取镜像时走的是https，所以需要配置下https，一下是默认用ip地址去配置的

```
# 所有操作都是在临时文件夹 ~/ssl 下操作
mkdir ~/ssl 

# 生成生成CA证书私钥
openssl genrsa -out ca.key 4096

# 生成CA证书
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Shenzhen/L=Shenzhen/O=example/OU=Personal/CN=192.168.50.21" \
 -key ca.key \
 -out ca.crt
 
 # 生成服务器证书-----生成私钥
 openssl genrsa -out 192.168.50.21.key 4096
 
 
 # 生成服务器证书-----生成证书签名请求（CSR）
 openssl req -sha512 -new \
    -subj "/C=CN/ST=Shenzhen/L=Shenzhen/O=example/OU=Personal/CN=192.168.50.21" \
    -key 192.168.50.21.key \
    -out 192.168.50.21.csr


# 生成服务器证书-----生成一个x509 v3扩展文件
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = IP:192.168.50.21
EOF


# 生成服务器证书-----使用该v3.ext文件为您的Harbor主机生成证书
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in 192.168.50.21.csr \
    -out 192.168.50.21.crt


# 将服务器证书和密钥复制到Harbor主机上的/data/cert/文件夹中
mkdir -p /data/cert/
cp 192.168.50.21.crt /data/cert/
cp 192.168.50.21.key /data/cert/


# 转换192.168.50.21.crt为192.168.50.21.cert，供Docker使用
openssl x509 -inform PEM -in 192.168.50.21.crt -out 192.168.50.21.cert


# 将服务器证书，密钥和CA文件复制到Harbor主机上的Docker证书文件夹中。您必须首先创建适当的文件夹
# 这个是使用4433端口，不用443端口的操作
mkdir -p /etc/docker/certs.d/192.168.50.21:4433/
cp 192.168.50.21.cert /etc/docker/certs.d/192.168.50.21:4433/
cp 192.168.50.21.key /etc/docker/certs.d/192.168.50.21:4433/
cp ca.crt /etc/docker/certs.d/192.168.50.21:4433/

#如果将默认nginx端口443 映射到其他端口，请创建文件夹
#/etc/docker/certs.d/yourdomain.com:port或/etc/docker/certs.d/harbor_IP:port


```

harbor用域名的形式配置

```
# 所有操作都是在临时文件夹 ~/ssl 下操作
mkdir ~/ssl 

# 生成生成CA证书私钥
openssl genrsa -out ca.key 4096

# 生成CA证书
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Shenzhen/L=Shenzhen/O=example/OU=Personal/CN=域名" \
 -key ca.key \
 -out ca.crt
 
 # 生成服务器证书-----生成私钥
 openssl genrsa -out 域名.key 4096
 
 
 # 生成服务器证书-----生成证书签名请求（CSR）
 openssl req -sha512 -new \
    -subj "/C=CN/ST=Shenzhen/L=Shenzhen/O=example/OU=Personal/CN=域名" \
    -key 域名.key \
    -out 192.168.50.21.csr


# 生成服务器证书-----生成一个x509 v3扩展文件
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = IP:192.168.50.21
EOF


# 生成服务器证书-----使用该v3.ext文件为您的Harbor主机生成证书
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in 域名.csr \
    -out 域名.crt


# 将服务器证书和密钥复制到Harbor主机上的/data/cert/文件夹中
mkdir -p /data/cert/
cp 域名.crt /data/cert/
cp 域名.key /data/cert/


# 转换域名.crt为域名.cert，供Docker使用
openssl x509 -inform PEM -in 域名.crt -out 域名.cert


# 将服务器证书，密钥和CA文件复制到Harbor主机上的Docker证书文件夹中。您必须首先创建适当的文件夹
# 这个是使用4433端口，不用443端口的操作
mkdir -p /etc/docker/certs.d/域名:4433/
cp 192.168.50.21.cert /etc/docker/certs.d/域名:4433/
cp 192.168.50.21.key /etc/docker/certs.d/域名:4433/
cp ca.crt /etc/docker/certs.d/域名:4433/

#如果将默认nginx端口443 映射到其他端口，请创建文件夹
#/etc/docker/certs.d/yourdomain.com:port或/etc/docker/certs.d/harbor_IP:port


```

修改harbor.yml

```
mv harbor.yml.tmpl harbor.yml
vim harbor.yml
```

```yml
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: 192.168.50.21

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 8099

# https related config
https:
  # https port for harbor, default is 443
  port: 4433
  # The path of cert and key files for nginx
  certificate: /data/cert/192.168.50.21.crt
  private_key: /data/cert/192.168.50.21.key

# # Uncomment following will enable tls communication between all harbor components
# internal_tls:
#   # set enabled to true means internal tls is enabled
#   enabled: true
#   # put your cert and key files on dir
#   dir: /etc/harbor/tls/internal

# Uncomment external_url if you want to enable external proxy
# And when it enabled the hostname will no longer used
# external_url: https://reg.mydomain.com:8433

# The initial password of Harbor admin
# It only works in first time to install harbor
# Remember Change the admin password from UI after launching Harbor.
harbor_admin_password: admin123

# Harbor DB configuration
database:
  # The password for the root user of Harbor DB. Change this before any production use.
  password: root123
  # The maximum number of connections in the idle connection pool. If it <=0, no idle connections are retained.
  max_idle_conns: 100
  # The maximum number of open connections to the database. If it <= 0, then there is no limit on the number of open connections.
  # Note: the default number of connections is 1024 for postgres of harbor.
  max_open_conns: 900

# The default data volume
data_volume: /data

# Harbor Storage settings by default is using /data dir on local filesystem
# Uncomment storage_service setting If you want to using external storage
# storage_service:
#   # ca_bundle is the path to the custom root ca certificate, which will be injected into the truststore
#   # of registry's and chart repository's containers.  This is usually needed when the user hosts a internal storage with self signed certificate.
#   ca_bundle:

#   # storage backend, default is filesystem, options include filesystem, azure, gcs, s3, swift and oss
#   # for more info about this configuration please refer https://docs.docker.com/registry/configuration/
#   filesystem:
#     maxthreads: 100
#   # set disable to true when you want to disable registry redirect
#   redirect:
#     disabled: false

# Trivy configuration
#
# Trivy DB contains vulnerability information from NVD, Red Hat, and many other upstream vulnerability databases.
# It is downloaded by Trivy from the GitHub release page https://github.com/aquasecurity/trivy-db/releases and cached
# in the local file system. In addition, the database contains the update timestamp so Trivy can detect whether it
# should download a newer version from the Internet or use the cached one. Currently, the database is updated every
# 12 hours and published as a new release to GitHub.
trivy:
  # ignoreUnfixed The flag to display only fixed vulnerabilities
  ignore_unfixed: false
  # skipUpdate The flag to enable or disable Trivy DB downloads from GitHub
  #
  # You might want to enable this flag in test or CI/CD environments to avoid GitHub rate limiting issues.
  # If the flag is enabled you have to download the `trivy-offline.tar.gz` archive manually, extract `trivy.db` and
  # `metadata.json` files and mount them in the `/home/scanner/.cache/trivy/db` path.
  skip_update: false
  #
  # The offline_scan option prevents Trivy from sending API requests to identify dependencies.
  # Scanning JAR files and pom.xml may require Internet access for better detection, but this option tries to avoid it.
  # For example, the offline mode will not try to resolve transitive dependencies in pom.xml when the dependency doesn't
  # exist in the local repositories. It means a number of detected vulnerabilities might be fewer in offline mode.
  # It would work if all the dependencies are in local.
  # This option doesn’t affect DB download. You need to specify "skip-update" as well as "offline-scan" in an air-gapped environment.
  offline_scan: false
  #
  # insecure The flag to skip verifying registry certificate
  insecure: false
  # github_token The GitHub access token to download Trivy DB
  #
  # Anonymous downloads from GitHub are subject to the limit of 60 requests per hour. Normally such rate limit is enough
  # for production operations. If, for any reason, it's not enough, you could increase the rate limit to 5000
  # requests per hour by specifying the GitHub access token. For more details on GitHub rate limiting please consult
  # https://developer.github.com/v3/#rate-limiting
  #
  # You can create a GitHub token by following the instructions in
  # https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line
  #
  # github_token: xxx

jobservice:
  # Maximum number of job workers in job service
  max_job_workers: 10

notification:
  # Maximum retry count for webhook job
  webhook_job_max_retry: 10

chart:
  # Change the value of absolute_url to enabled can enable absolute url in chart
  absolute_url: disabled

# Log configurations
log:
  # options are debug, info, warning, error, fatal
  level: info
  # configs for logs in local storage
  local:
    # Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
    rotate_count: 50
    # Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
    # If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
    # are all valid.
    rotate_size: 200M
    # The directory on your host that store log
    location: /var/log/harbor

  # Uncomment following lines to enable external syslog endpoint.
  # external_endpoint:
  #   # protocol used to transmit log to external endpoint, options is tcp or udp
  #   protocol: tcp
  #   # The host of external endpoint
  #   host: localhost
  #   # Port of external endpoint
  #   port: 5140

#This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
_version: 2.5.0

# Uncomment external_database if using external database.
# external_database:
#   harbor:
#     host: harbor_db_host
#     port: harbor_db_port
#     db_name: harbor_db_name
#     username: harbor_db_username
#     password: harbor_db_password
#     ssl_mode: disable
#     max_idle_conns: 2
#     max_open_conns: 0
#   notary_signer:
#     host: notary_signer_db_host
#     port: notary_signer_db_port
#     db_name: notary_signer_db_name
#     username: notary_signer_db_username
#     password: notary_signer_db_password
#     ssl_mode: disable
#   notary_server:
#     host: notary_server_db_host
#     port: notary_server_db_port
#     db_name: notary_server_db_name
#     username: notary_server_db_username
#     password: notary_server_db_password
#     ssl_mode: disable

# Uncomment external_redis if using external Redis server
# external_redis:
#   # support redis, redis+sentinel
#   # host for redis: <host_redis>:<port_redis>
#   # host for redis+sentinel:
#   #  <host_sentinel1>:<port_sentinel1>,<host_sentinel2>:<port_sentinel2>,<host_sentinel3>:<port_sentinel3>
#   host: redis:6379
#   password: 
#   # sentinel_master_set must be set to support redis+sentinel
#   #sentinel_master_set:
#   # db_index 0 is for core, it's unchangeable
#   registry_db_index: 1
#   jobservice_db_index: 2
#   chartmuseum_db_index: 3
#   trivy_db_index: 5
#   idle_timeout_seconds: 30

# Uncomment uaa for trusting the certificate of uaa instance that is hosted via self-signed cert.
# uaa:
#   ca_file: /path/to/ca

# Global proxy
# Config http proxy for components, e.g. http://my.proxy.com:3128
# Components doesn't need to connect to each others via http proxy.
# Remove component from `components` array if want disable proxy
# for it. If you want use proxy for replication, MUST enable proxy
# for core and jobservice, and set `http_proxy` and `https_proxy`.
# Add domain to the `no_proxy` field, when you want disable proxy
# for some special registry.
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy

# metric:
#   enabled: false
#   port: 9090
#   path: /metrics

# Trace related config
# only can enable one trace provider(jaeger or otel) at the same time,
# and when using jaeger as provider, can only enable it with agent mode or collector mode.
# if using jaeger collector mode, uncomment endpoint and uncomment username, password if needed
# if using jaeger agetn mode uncomment agent_host and agent_port
# trace:
#   enabled: true
#   # set sample_rate to 1 if you wanna sampling 100% of trace data; set 0.5 if you wanna sampling 50% of trace data, and so forth
#   sample_rate: 1
#   # # namespace used to differenciate different harbor services
#   # namespace:
#   # # attributes is a key value dict contains user defined attributes used to initialize trace provider
#   # attributes:
#   #   application: harbor
#   # # jaeger should be 1.26 or newer.
#   # jaeger:
#   #   endpoint: http://hostname:14268/api/traces
#   #   username:
#   #   password:
#   #   agent_host: hostname
#   #   # export trace data by jaeger.thrift in compact mode
#   #   agent_port: 6831
#   # otel:
#   #   endpoint: hostname:4318
#   #   url_path: /v1/traces
#   #   compression: false
#   #   insecure: true
#   #   timeout: 10s

# enable purge _upload directories
upload_purging:
  enabled: true
  # remove files in _upload directories which exist for a period of time, default is one week.
  age: 168h
  # the interval of the purge operations
  interval: 24h
  dryrun: false
```

harbor登录

```
sz-memect.tpddns.cn:4433
账号：admin
密码：admin123
```

harbor卸载

```
 rm -rf `find / -name harbor`
 删除容器
 docker rmi $(docker images -q)
```

另外一台机器拉取和推送镜像

```
mkdir /etc/docker/certs.d/192.168.50.21:4433
```

然后后从 Harbor 主机拷贝证书文件到 Docker 客户端上，需要 server 的证书和密钥以及 CA 证书。

```
scp /data/cert/192.168.50.21.key  root@192.168.50.20:/etc/docker/certs.d/192.168.50.21:4433
scp /data/cert/192.168.50.21.cert  root@192.168.50.20:/etc/docker/certs.d/192.168.50.21:4433
scp /data/cert/ca.crt  root@192.168.50.20:/etc/docker/certs.d/192.168.50.21:4433
```

拉取命令

```
docker pull 192.168.50.21:4433/library/mysql:latest
```

推送镜像

```
# 打tag
docker tag mongo:latest 192.168.50.21:4433/library/mongo:latest

# 推送镜像
docker push 192.168.50.21:4433/library/mongo:latest
```

