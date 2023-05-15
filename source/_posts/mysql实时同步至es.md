---
title: mysql实时同步至es
date: 2021-11-20 12:31:01
tags: mysql
categories: 学习
keywords: mysql
description: mysql实时同步
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---


# mysql实时同步至es

目前是在本地进行测试同步的，由于在服务器上运行，占用的内存有点大，服务器上的配置与本地的配置稍微有所差别

使用的工具版本：

MySql8.0：通过 show variables like '%log_bin%'; 查看*二进制日志功能是否开启，一般情况下是默认开启的*

> **Elasticsearch7.10.2由于log4j的问题导致用docker安装不了，可以换个版本**

| 工具           | 端口  | 版本   | 问题                                                         |
| -------------- | ----- | ------ | ------------------------------------------------------------ |
| mysql          | 3308  | 5.7    |                                                              |
| Elasticsearch  | 9200  | 7.10.2 | [7.17.0](https://hub.docker.com/layers/elasticsearch/library/elasticsearch/7.17.0/images/sha256-fa7141154a7e14df214e42f08c333702403eb88c02ba44e79322a1f42d733013?context=explore) |
| Kibanba        | 5601  | 7.10.2 |                                                              |
| canal-deployer | 11111 | 1.1.16 |                                                              |
| canal-adapter  | 8081  | 1.1.16 |                                                              |
| canal-admin    | 8089  | 1.1.16 |                                                              |

# 1.安装MySql

通过docker安装mysql，pull下来mysql5.7版本

docker pull mysql:5.7

新建目录及配置文件

```sh
cd /Users/mac/  
mkdir mysql  
cd mysql  
mkdir {data,conf,logs}  
cd conf  
vim my.cnf
```

创建并修改my.cnf配置文件

```cnf
[mysqld]  
## 设置server_id，同一局域网中需要唯一  
server_id=101  
## 指定不需要同步的数据库名称  
binlog-ignore-db=mysql    
## 开启二进制日志功能  
log-bin=mall-mysql-bin    
## 设置二进制日志使用内存大小（事务）  
binlog_cache_size=1M    
## 设置使用的二进制日志格式（mixed,statement,row）  
binlog_format=row    
## 二进制日志过期清理时间。默认值为0，表示不自动清理。  
expire_logs_days=7    
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。  
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致  
slave_skip_errors=1062    

## 增加配置
character-set-server=utf8  
secure_file_priv=/var/lib/mysql
```

通过docker启动mysql5.7

```sh
docker run -p 3308:3306 --name mysql57 -v /Users/mac/canal/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /Users/mac/canal/mysql/logs:/logs -v /Users/mac/canal/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

启动后，通过docker ps查看mysql是否启动成功。

如果启动成功了，那么运行

```sh
docker exec -it 容器id /bin/bash
```

进入容器查看，通过输入命令进入mysql

```
mysql -uroot -p
```

输入密码：123456进行mysql，之后可以通过输入查询语句，查看my.cnf是否配置成功。

这是查看binlog是否启用

```
show variables like '%log_bin%';
```

![image-20221010105441843](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010105441843.png)

这是查看binlog模式

```
show variables like 'binlog_format%';  
```

![image-20221010105454917](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010105454917.png)

建立一个测试用的数据库，canal_test数据库（这里是直接可视化界面建立的），之后再建立一张商品表product

```sql
CREATE TABLE `product`  (  
  `id` bigint(20) NOT NULL AUTO_INCREMENT,  
  `title` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,  
  `sub_title` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,  
  `price` decimal(10, 2) NULL DEFAULT NULL,  
  `pic` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,  
  PRIMARY KEY (`id`) USING BTREE  
) ENGINE = InnoDB AUTO_INCREMENT = 2 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```



# 2.Elasticsearch和Kibana安装

上dockerhub上，找到es的版本，然后下载安装，在mac上安装和通过docker安装并没有什么太大的区别

docker pull elasticsearch:7.17.0

之后运行es，如果没有映射文件夹，记得创建

```sh
docker run --name myes -d -e ES_JAVA_OPTS="-Xms256m -Xmx256m"   
-e "discovery.type=single-node" -p 9200:9200 -p 9300:9300   
-v /etc/localtime:/etc/localtime:ro   
-v /home/es/data:/usr/share/elasticsearch/data elasticsearch:7.17.0
```

同理，kibana也是这样安装的。拉取镜像，创建配置文件夹，

```
docker pull kibana:7.17.0

mkdir -p /root/mydata/kibana/conf  
cd /root/mydata/kibana/conf
```

**vim kibana.yml**  这是kibana的配置文件，配置连接es的

```
# 
## ** THIS IS AN AUTO-GENERATED FILE **  
#  
# Default Kibana configuration for docker target  

server.name: kibana  
server.host: "0"  
```

如果是云服务器，则将127.0.0.1改成服务器的ip地址  

```
elasticsearch.hosts: [ "http://127.0.0.1:9200" ]  
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

之后是创建kibana容器

```
docker run -d --name=kibana --restart=always -p 5601:5601   
-v $PWD/conf/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.17.0
```



# 3.canal组件安装

## 3.1安装canal-server

-   首先我们需要下载canal的各个组件canal-server、canal-adapter、canal-admin，下载地址：<https://github.com/alibaba/canal/releases>

-   **canal是基于Java的，启动canal服务器一定要保证安装好了jdk**

![image-20221010105719713](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010105719713.png)

canal的各个组件的用途各不相同，下面分别介绍下：

-   **canal-server（canal-deploy）**：可以直接监听MySQL的binlog，把自己伪装成MySQL的从库，只负责接收数据，并不做处理。

-   canal-adapter：相当于canal的客户端，会从canal-server中获取数据，然后对数据进行同步，可以同步到MySQL、Elasticsearch和HBase等存储中去。

-   canal-admin：为canal提供整体配置管理、节点运维等面向运维的功能，提供相对友好的WebUI操作界面，方便更多用户快速和安全的操作。

进入到canal组件目录，然后将他们分别解压至相应的目录

```
cd /Users/mac/canal  
mkdir {canal-server,canal-adapter,canal-admin}  
tar -zxvf canal.deployer-1.1.6-SNAPSHOT.tar.gz -C canal-server  
tar -zxvf canal.adapter-1.1.6-SNAPSHOT.tar.gz -C canal-adapter  
tar -zxvf canal.admin-1.1.6-SNAPSHOT.tar.gz -C canal-admin
```

进入到canal-server目录，修改配置文件**conf/example/instance.properties**，按照如下配置

```
# 需要同步数据的MySQL地址  
canal.instance.master.address=127.0.0.1:3308  
canal.instance.master.journal.name=  
canal.instance.master.position=  
canal.instance.master.timestamp=  
canal.instance.master.gtid=  
# 用于同步数据的数据库账号  
canal.instance.dbUsername=root  
# 用于同步数据的数据库密码  
canal.instance.dbPassword=123456  
# 数据库连接编码  
canal.instance.connectionCharset = UTF-8  
# 需要订阅binlog的表过滤正则表达式  
canal.instance.filter.regex=.*\\..*
```

-   使用startup.sh脚本启动canal-server服务；

```
sh bin/startup.sh
```

-   启动成功后可使用如下命令查看服务日志信息；

-   注意：只有启动成功了，才会创建canal文件夹和cancal/canal.log、example文件夹和example/example.log 

```
tail -f logs/canal/canal.log
```

![image-20221010105823795](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010105823795.png)

-   启动成功后可使用如下命令查看instance日志信息；

```
tail -f logs/example/example.log 
```

![image-20221010105837062](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010105837062.png)

-   如果想要停止canal-server服务可以使用如下命令。

```
sh bin/stop.sh
```



## 3.2安装canal-adapter

进入canal-adapter目录

```
cd /Users/mac/canal/canal-adapter
```



-   修改配置文件conf/application.yml，按如下配置即可，主要是修改canal-server配置、数据源配置和客户端适配器配置；

  - - ```
       canal.conf:  
         mode: tcp # 客户端的模式，可选tcp kafka rocketMQ  
         flatMessage: true # 扁平message开关, 是否以json字符串形式投递数据, 仅在kafka/rocketMQ模式下有效  
         zookeeperHosts:    # 对应集群模式下的zk地址  
         syncBatchSize: 1000 # 每次同步的批数量  
         retries: 0 # 重试次数, -1为无限重试  
         timeout: # 同步超时时间, 单位毫秒  
         accessKey:  
         secretKey:  
         consumerProperties:  
       
           # canal tcp consumer  
       
           canal.tcp.server.host: 127.0.0.1:11111 #设置canal-server的地址  
           canal.tcp.zookeeper.hosts:  
           canal.tcp.batch.size: 500  
           canal.tcp.username:  
           canal.tcp.password:  
       
         srcDataSources: # 源数据库配置  
           defaultDS:  
             url: jdbc:mysql://127.0.0.1:3308/canal_test?useUnicode=true  
             username: root  
             password: 123456  
         canalAdapters: # 适配器列表  
       
         - instance: example # canal实例名或者MQ topic名  
             groups: # 分组列表  
              - groupId: g1 # 分组id, 如果是MQ模式将用到该值  
                outerAdapters:  
                   - name: logger # 日志打印适配器  
                     name: es7 # ES同步适配器  
                             hosts: http://127.0.0.1:9200 # ES连接地址  
                             properties:  
                               mode: rest # 模式可选transport(9300) 或者 rest(9200)  
                               # security.auth: test:123456 #  only used for rest mode  
                               cluster.name: elasticsearch # ES集群名称
       ```
  
       
  
-   注意：在配置es时，es的连接地址前面一定要加http://，不然在启动canal-adapter组件时就会报错；还有就是: 后面一定要加空格

-   **注意：如果是在云服务上启动报错，错误内容为**

    -   Could not resolve placeholder 'HOSTNAME%%.*'  
        in value "history -a; printf "033]0;%s@%s:%s007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}""

    -   那么，则需要在application.yml的配置文件中最后面定格加入

    -   HOSTNAME%%.*:  
        PWD/*#$HOME/~:*

![image-20221010105931138](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010105931138.png)

-   可以参考链接：[启动canal-adapter时候未成功](https://github.com/alibaba/canal/issues/3063)&nbsp;

-   添加配置文件canal-adapter/conf/es7/product.yml，用于配置MySQL中的表与Elasticsearch中索引的映射关系；

```
dataSourceKey: defaultDS # 源数据源的key, 对应上面配置的srcDataSources中的值  
destination: example  # canal的instance或者MQ的topic  
groupId: g1 # 对应MQ模式下的groupId, 只会同步对应groupId的数据  
esMapping:  
  _index: canal_product # es 的索引名称  
  _id: _id  # es 的_id, 如果不配置该项必须配置下面的pk项_id则会由es自动分配  
  sql: "SELECT  
            p.id AS _id,  
            p.title,  
            p.sub_title,  
            p.price,  
            p.pic  
        FROM  
            product p"        # sql映射  
  etlCondition: "where a.c_time&gt;={}"   #etl的条件参数  
  commitBatch: 3000   # 提交批大小
```

-   使用startup.sh脚本启动canal-adapter服务；

```
sh bin/startup.sh
```

-   启动成功后可使用如下命令查看服务日志信息；

```
tail -f logs/adapter/adapter.log
```

![image-20221010110018421](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010110018421.png)

-   启动成功后，日志会将表的映射文件信息给打印出来

-   如果需要停止canal-adapter服务可以使用如下命令。

-   sh bin/stop.sh

# 4.数据同步测试

-   首先我们需要在Elasticsearch中创建索引，和MySQL中的product表相对应，直接在Kibana的Dev Tools中使用如下命令创建即可；

```
PUT canal_product  
{  
  "mappings": {  
    "properties": {  
      "title": {  
        "type": "text"  
      },  
      "sub_title": {  
        "type": "text"  
      },  
      "pic": {  
        "type": "text"  
      },  
      "price": {  
        "type": "double"  
      }  
    }  
  }  
}
```

![image-20221010110038451](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010110038451.png)

-   创建完成后可以查看下索引的结构；

GET canal_product/_mapping

![image-20221010110052888](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010110052888.png)

-   之后使用如下SQL语句在数据库中创建一条记录；

```sql
INSERT INTO product ( id, title, sub_title, price, pic ) VALUES ( 5, '小米8', ' 全面屏游戏智能手机 6GB+64GB', 1999.00, NULL );
```

-   创建成功后，在Elasticsearch中搜索下，发现数据已经同步了；

GET canal_product/_search

![image-20221010110126304](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010110126304.png)

-   再使用如下SQL对数据进行修改；

```sql
UPDATE product SET title='小米10' WHERE id=5
```

-   修改成功后，在Elasticsearch中搜索下，发现数据已经修改了；

![image-20221010110145988](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010110145988.png)

-   再使用如下SQL对数据进行删除操作；

```sql
DELETE FROM product WHERE id=5
```

-   删除成功后，在Elasticsearch中搜索下，发现数据已经删除了，至此MySQL同步到Elasticsearch的功能完成了！

![image-20221010110213694](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/image-20221010110213694.png)
