---
title: 使用docker-compose部署单机MySQL
description: 本文讲述如何使用docker-compose部署单机MySQL8
date: 2021-03-02 20:15:03
tags: 
  - docker
  - mysql
categories:
  - docker

---


### 拉取MySQL镜像

```bash
docker pull mysql:8
```

### 创建挂载目录

```bash
mkdir -p /app/docker/mysql8/log
mkdir -p /app/docker/mysql8/data
```

### 编写 docker-compose.yml 文件
```bash
cd /app/docker/mysql8
vim docker-compose.yml
```

```yml
services:
  mysql:
    container_name: mysql
    image: mysql:8
    restart: always
    ports:
      - "3306:3306"
    environment:
      TZ: Asia/Shanghai # 设置容器时区
      MYSQL_TCP_PORT: 3306
      MYSQL_ROOT_PASSWORD: root # root用户密码
      #MYSQL_ROOT_HOST: 'localhost' # 是否开启root账号的远程登录,默认是开启的,启用:填'%'或者注释掉这行,禁用:填'localhost'
    volumes:
      - "./data:/var/lib/mysql" # 映射数据目录
      - "./log:/var/log/mysql" # 映射日志目录
```

### 启动容器

```bash
docker compose up -d
```

### 运行后查看启动容器的情况

```bash
docker ps
docker compose logs 
```

### 连接测试

```bash
docker exec -it 容器ID /bin/bash
mysql -h127.0.0.1 -uroot -p
输入密码
```

如果远程连接不上可以看看防火墙是否已经开放3306端口

以Ubuntu为例:

```bash
sudo ufw status
sudo ufw status numbered
sudo ufw allow 3306
```