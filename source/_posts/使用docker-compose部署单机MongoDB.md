---
title: 使用docker-compose部署单机MongoDB
description: 本文讲述如何使用docker-compose部署单机MongoDB5
date: 2021-08-20 20:15:03
tags:
  - docker
  - mongo
  
categories:
  - docker

---

### 创建挂载目录

```bash
mkdir -p /app/docker/mongo/data/db
```

### 编写 docker-compose.yml 文件
```bash
cd /app/docker/mongo
vim docker-compose.yml
```

```yml
services:
  mongo:
    container_name: mongo
    image: mongo:5
    restart: always
    ports:
      - "27017:27017"
    environment:
      TZ: Asia/Shanghai
      MONGO_INITDB_ROOT_USERNAME: root  # 配置了这两个参数后,就会以mongo --auth开启认证模式启动,并且creating a simple user with the role root in the admin authentication database
      MONGO_INITDB_ROOT_PASSWORD: 123456
    volumes:
      - "./data/db:/data/db"

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

#mongo cli 连接测试
docker run -it --network mongo_default --rm mongo:5 mongosh --host mongo -u root -p 123456 --authenticationDatabase admin test
db.getName();
```