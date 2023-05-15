---
title: Pritunl安装
date: 2021-11-20 12:31:01
tags: Pritunl
categories: 安装
keywords: pritunl
description: pritunl安装
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# Pritunl的安装

- 环境准备

  | 序号 | 环境准备       | 版本                     |
  | ---- | -------------- | ------------------------ |
  | 1    | Ubantu         | 20.04桌面版              |
  | 2    | Pritunl Server | Pritunl Server最新的版本 |
  | 3    | Pritunl Client | Pritunl Client最新的版本 |

  

- Pritunl Server安装

  1、通过`apt`在终端中运行以下命令来确保所有系统软件包都是最新的

  ```
  sudo apt update
  sudo apt upgrade
  sudo apt install curl gnupg2 wget unzip
  ```

  2、在Ubuntu 20.04上安装Pritunl VPN Server

  我们将Pritunl存储库密钥和文件添加到Ubuntu系统

  ```
  sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv E162F504A20CDF15827F718D4B7C549A058F8B6B
  sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A
  ```

  接下来，使用以下命令添加其存储库文件：

  ```
  echo "deb http://repo.pritunl.com/stable/apt focal main" | sudo tee /etc/apt/sources.list.d/pritunl.list
  ```

  添加存储库后，请使用以下命令更新存储库缓存并安装Pritunl服务器：

  ```
  sudo apt update
  sudo apt install pritunl
  ```

  安装完成后，您可以设置启动，停止并启用Pritunl VPN服务器在服务器启动时自动启动：

  ```
  sudo systemctl stop pritunl
  sudo systemctl start pritunl
  sudo systemctl enable pritunl
  ```

  3、安装MongoDB

  默认情况下，Ubuntu 20.04默认存储库中不提供MongoDB，因此您需要将MongoDB存储库添加到系统中：

  ```
  curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
  echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
  ```

  `apt`使用以下命令更新和安装MongoDB：

  ```
  sudo apt update
  sudo apt-get install mongodb-server
  ```

  4、访问Pritunl

  现在，打开您的Web浏览器，并使用URL访问Pritunl Web界面。应该看到以下屏幕：`https://your-server-ipaddress`

  ![pritunl-web-interface](https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora/pritunl-web-interface.jpg)

​		接下来，使用以下命令生成设置密钥：

```
sudo pritunl setup-key
```

​		输出：

```
from cryptography import x586
c879d9o59b10012084887b2g4ua5g49u
```

之后，复制安装密钥并将其粘贴到Pritunl数据库安装向导。粘贴设置键后，单击“保存”按钮。

接下来，您将需要生成一个默认的用户名和密码来登录。为此，请运行以下命令：

```
sudo pritunl default-password
```

输出：

```
from cryptography import x586
[undefined][2021-03-09 12:09:16,430][INFO] Getting default administrator password
Administrator default password:
username: "pritunl"
password: "meilana123"
```

现在，使用默认的用户名和密码登录，并从Pritunl仪表板设置您的环境。

- Pritunl Clinet安装（只能在Ubantu桌面上安装，安装后可以在Ubantu系统上看见软件图标）

  ```
  sudo tee /etc/apt/sources.list.d/pritunl.list << EOF
  deb https://repo.pritunl.com/stable/apt focal main
  EOF
  
  sudo apt --assume-yes install gnupg
  gpg --keyserver hkp://keyserver.ubuntu.com --recv-keys 7568D9BB55FF9E5287D586017AE645C0CF8E292A
  gpg --armor --export 7568D9BB55FF9E5287D586017AE645C0CF8E292A | sudo tee /etc/apt/trusted.gpg.d/pritunl.asc
  sudo apt update
  sudo apt install pritunl-client-electron
  ```

  