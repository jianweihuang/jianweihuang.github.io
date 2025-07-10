---
title: 使用docker-compose部署单机Redis
description: 本文讲述如何使用docker-compose部署单机Redis
tags:
  - docker
  - redis
categories:
  - docker
date: 2021-03-02 20:35:03
---

### 创建挂载目录

```bash
mkdir -p /app/docker/redis/conf
mkdir -p /app/docker/redis/data
```

### 编写docker-compose.yml文件和redis.conf配置文件

docker-compose.yml文件

```bash
cd /app/docker/redis
vim docker-compose.yml
```

```yml
services:
  redis:
    image: redis:6
    container_name: redis
    restart: always
    ports:
      - 6379:6379
    environment:
      TZ: Asia/Shanghai
    volumes:
      - ./conf:/etc/redis
      - ./data:/data
    command:
      redis-server /etc/redis/redis.conf
```

配置文件 redis.conf

详细redis.conf可参考官方文档:
[https://redis.io/docs/latest/operate/oss_and_stack/management/config/](https://redis.io/docs/latest/operate/oss_and_stack/management/config/)

```bash
cd /app/docker/redis/conf
vim redis.conf
```

```
# Redis 服务器的端口号（默认：6379）
port 6379

# 绑定的 IP 地址，如果设置为 127.0.0.1，则只能本地访问；若设置为 0.0.0.0，则监听所有接口（默认：127.0.0.1）
bind 0.0.0.0

# 设置密码，客户端连接时需要提供密码才能进行操作，如果不设置密码，可以注释掉此行（默认：无）
# requirepass foobared
requirepass 123456

# 设置在客户端闲置一段时间后关闭连接，单位为秒（默认：0，表示禁用）
timeout 0

# 是否以守护进程（daemon）模式运行，默认为 "no"，设置为 "yes" 后 Redis 会在后台运行
daemonize no

# 设置日志级别（默认：notice）。可以是 debug、verbose、notice、warning
loglevel notice

# 设置日志文件的路径（默认：空字符串），如果不设置，日志会输出到标准输出
logfile ""

# 设置数据库数量（默认：16），Redis 使用数据库索引从 0 到 15
databases 16

# 是否启用 AOF 持久化，默认为 "no"。如果设置为 "yes"，将在每个写操作执行时将其追加到文件中
appendonly yes

# 设置 AOF 持久化的文件路径（默认：appendonly.aof）
appendfilename "appendonly.aof"

# AOF 持久化模式，默认为 "always"。可以是 always、everysec 或 no
# always：每个写操作都立即同步到磁盘
# everysec：每秒钟同步一次到磁盘
# no：完全依赖操作系统的行为，可能会丢失数据，但性能最高
# appendfsync always

save 3600 1
save 300 100
save 60 10000

# 设置 RDB 持久化文件的名称（默认：dump.rdb）
# dbfilename dump.rdb

# 设置是否开启慢查询日志，默认为 "no"
# slowlog-log-slower-than 10000

# 设置慢查询日志的最大长度，默认为 128
# slowlog-max-len 128

# 设置每秒最大处理的写入命令数量，用于保护 Redis 服务器不被超负荷写入（默认：0，表示不限制）
# maxclients 10000

# 设置最大连接客户端数量（默认：10000，0 表示不限制）
# maxmemory <bytes>

# 设置最大使用内存的策略（默认：noeviction）。可以是 volatile-lru、allkeys-lru、volatile-random、allkeys-random、volatile-ttl 或 noeviction
# maxmemory-policy noeviction

# 设置允许最大使用内存的比例（默认：0），设置为 0 表示禁用
# maxmemory-samples 5

# 设置 RDB 持久化文件的保存路径，默认保存在当前目录
# dir ./

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
docker exec -it redis bash
redis-cli -h 127.0.0.1 -p 6379
auth 123456
ping
```

如果一切正常，你会看到Redis响应 "PONG" ，表示连接成功。
