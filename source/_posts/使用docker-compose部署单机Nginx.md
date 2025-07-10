---
title: 使用docker-compose部署单机Nginx
description: 本文讲述如何使用docker-compose部署单机Nginx
date: 2021-05-18 21:19:20
tags:
  - docker
  - nginx

categories:
  - docker

---

### 创建挂载目录

```bash
mkdir -p /app/docker/nginx/conf/conf.d
mkdir -p /app/docker/nginx/html
mkdir -p /app/docker/nginx/log
```

### 编写 docker-compose.yml 文件

```bash
cd /app/docker/nginx
vim docker-compose.yml
```

```yml
services:
  nginx:
    container_name: nginx
    image: nginx:1.20.0
    restart: always
    ports:
      - "80:80"
      - "443:443"
    environment:
      TZ: Asia/Shanghai
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./conf/conf.d:/etc/nginx/conf.d
      - ./log:/var/log/nginx
      - ./html:/usr/share/nginx/html:ro

```

### 编写配置文件 nginx.conf 文件

注:此文件为nginx的默认配置文件复制而来,可以自己修改其中的配置

```bash
cd /app/docker/nginx/conf
vim nginx.conf
```

```
user  nginx;
# nginx 进程数，建议按照cpu 数目来指定，一般为它的倍数 (如,2个四核的cpu计为8)
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    use epoll;
    worker_connections 51200;
    multi_accept on;
}


http {

    #include       mime.types;
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # 自定义日志格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
    # main格式记录访问日志
    #access_log  /var/log/nginx/access.log  main;
    # 不记录日志
    access_log off;

    client_max_body_size 50m;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    gzip on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types     text/plain application/javascript application/x-javascript text/javascript text/css application/xml;
    gzip_vary on;
    gzip_proxied   expired no-cache no-store private auth;
    gzip_disable   "MSIE [1-6]\.";

    server_tokens off;

    # 包含配置文件目录
    include /etc/nginx/conf.d/*.conf;
}

```

编写default.conf

```
cd /app/docker/nginx/conf/conf.d
vim default.conf
```

```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
        index  index.html;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}

```


### 编写myweb网站配置文件

```bash
cd /app/docker/nginx/conf/conf.d
vim www.myweb.com.conf
```

```
server {
    listen       80;
    server_name  www.myweb.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html/myweb;
        # 如果前端页面是Vue,需要添加try_files配置
        try_files $uri $uri/ /index.html;
        index index.html;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

}

```

### 部署html

```
mkdir -p /app/docker/nginx/html/myweb
cd /app/docker/nginx/html/myweb
vim index.html

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>myweb</title>
</head>
<body>
    <h1>Hello World!</h1>
</body>
</html>
```


### 启动容器

```bash
cd /app/docker/nginx
docker compose up -d
```

### 运行后查看启动容器的情况

```bash
docker ps
docker compose logs 
```

### 测试

SwitchHosts [https://github.com/oldj/SwitchHosts/releases](https://github.com/oldj/SwitchHosts/releases)

通过SwitchHosts临时添加hosts
```
127.0.0.1 www.myweb.com
```

浏览器访问: 
http://localhost:80
http://www.myweb.com

### 其他

```bash
# 重新加载配置
docker exec nginx nginx -s reload
```