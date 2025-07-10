---
title: 使用docker-compose部署单机influxdb
description: 本文讲述如何使用docker-compose部署单机influxdb
date: 2022-06-19 21:15:09
tags:
  - docker
  - influxdb

categories:
  - docker

---

### 创建挂载目录

```bash
mkdir -p /app/docker/influxdb/data
mkdir -p /app/docker/influxdb/config
```

### 编写 docker-compose.yml 文件
```bash
cd /app/docker/influxdb
vim docker-compose.yml
```

```yml
services:
  influxdb2:
    container_name: influxdb2
    image: influxdb:2
    restart: unless-stopped
    ports:
      - "8086:8086"
    environment:
      TZ: Asia/Shanghai
      DOCKER_INFLUXDB_INIT_MODE: setup
      DOCKER_INFLUXDB_INIT_USERNAME: root
      DOCKER_INFLUXDB_INIT_PASSWORD: root123456 # 密码不能太短,否则启动不成功
      DOCKER_INFLUXDB_INIT_ORG: myorg
      DOCKER_INFLUXDB_INIT_BUCKET: mybucket
    volumes:
      - "./data:/var/lib/influxdb2"
      - "./config:/etc/influxdb2"

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

### 访问控制台

浏览器打开: http://localhost:8086

