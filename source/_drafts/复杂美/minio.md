# MinIo



demo地址：[172.16.101.107:9000](https://confluence.33.cn/pages/172.16.101.107:9000)﻿

测试账号：minio

测试密码：minio123

可手动生成 ak，sk，bucket， 设置 bucket 权限（public）, 不支持临时角色



# 官方文档

英文文档：https://docs.min.io/﻿

中文文档：http://docs.minio.org.cn/docs/master/minio-client-complete-guide





# 搭建单节点 minio

```yaml
version: "3"
services:
  minio:
    image: minio/minio:RELEASE.2021-07-22T05-23-32Z
    container_name: dtalk_minio
    ports:
      - 9010:9000
      - 9011:9001
    volumes:
      - ./minio/data:/data
    command: server --console-address ":9001" /data
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    restart: always
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:9010/minio/health/live" ]
      interval: 30s
      timeout: 20s
      retries: 3

```



# 迁移数据

[MinIO 数据从旧集群迁移到新集群 - 简书 (jianshu.com)](https://www.jianshu.com/p/5c6bc2e3b886)
[Rclone结合MinIO Server | Minio中文文档](http://docs.minio.org.cn/docs/master/rclone-with-minio-server)



# 本地快速搭建 minio

目录

```
minio
├── docker-compose.yml
└── nginx.conf
```



docker-compose.yml

```yaml
version: '3.7'

# Settings and configurations that are common for all containers
x-minio-common: &minio-common
  image: minio/minio:RELEASE.2021-07-22T05-23-32Z
  command: server --console-address ":9001" http://minio{1...4}/data{1...2}
  expose:
    - "9000"
    - "9001"
  environment:
    MINIO_ROOT_USER: minio
    MINIO_ROOT_PASSWORD: minio123
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
    interval: 30s
    timeout: 20s
    retries: 3

# starts 4 docker containers running minio server instances.
# using nginx reverse proxy, load balancing, you can access
# it through port 9000.
services:
  minio1:
    <<: *minio-common
    hostname: minio1
    volumes:
      - data1-1:/data1
      - data1-2:/data2

  minio2:
    <<: *minio-common
    hostname: minio2
    volumes:
      - data2-1:/data1
      - data2-2:/data2

  minio3:
    <<: *minio-common
    hostname: minio3
    volumes:
      - data3-1:/data1
      - data3-2:/data2

  minio4:
    <<: *minio-common
    hostname: minio4
    volumes:
      - data4-1:/data1
      - data4-2:/data2

  nginx:
    image: nginx:1.19.2-alpine
    hostname: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "9000:9000"
      - "9001:9001"
    depends_on:
      - minio1
      - minio2
      - minio3
      - minio4

## By default this config uses default local driver,
## For custom volumes replace with volume driver configuration.
volumes:
  data1-1:
  data1-2:
  data2-1:
  data2-2:
  data3-1:
  data3-2:
  data4-1:
  data4-2:
```



nginx.conf

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  4096;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;

    # include /etc/nginx/conf.d/*.conf;

    upstream minio {
        server minio1:9000;
        server minio2:9000;
        server minio3:9000;
        server minio4:9000;
    }

    upstream console {
        ip_hash;
        server minio1:9001;
        server minio2:9001;
        server minio3:9001;
        server minio4:9001;
    }

    server {
        listen       9000;
        listen  [::]:9000;
        server_name  localhost;

        # To allow special characters in headers
        ignore_invalid_headers off;
        # Allow any size file to be uploaded.
        # Set to a value such as 1000m; to restrict file size to a specific value
        client_max_body_size 0;
        # To disable buffering
        proxy_buffering off;

        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 300;
            # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            chunked_transfer_encoding off;

            proxy_pass http://minio;
        }
    }

    server {
        listen       9001;
        listen  [::]:9001;
        server_name  localhost;

        # To allow special characters in headers
        ignore_invalid_headers off;
        # Allow any size file to be uploaded.
        # Set to a value such as 1000m; to restrict file size to a specific value
        client_max_body_size 0;
        # To disable buffering
        proxy_buffering off;

        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-NginX-Proxy true;

            # This is necessary to pass the correct IP to be hashed
            real_ip_header X-Real-IP;

            proxy_connect_timeout 300;
            # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            chunked_transfer_encoding off;

            proxy_pass http://console;
        }
    }
}
```



启动命令

```she
$ cd minio
$ docker-compose up -d
# 浏览器打开 127.0.0.1:9000 , 能进去后台界面说明启动成功
```



# minio Go SDK

[minio/minio-go: MinIO Client SDK for Go (github.com)](https://github.com/minio/minio-go/)



# 配置TLS

1. you can generate the SSL Certificate if you don't have one, for example:

   ```shell
   $ sudo mkdir -p /tmp/.minio/certs
   $ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/.minio/certs/private.key -out /tmp/.minio/certs/public.crt
   ```

2. run Minio sever secured by HTTPS. Here I'm using Docker with docker-compose:

   docker-compose.yaml:

   ```yaml
   version: '3'
   
   services:
     minio:
       image: minio/minio
       command: server --address ":443" /data
       ports:
         - "443:443"
       environment:
         MINIO_ACCESS_KEY: "YourAccesskey"
         MINIO_SECRET_KEY: "YourSecretkey"
       volumes:
         - /tmp/minio/data:/data
         - /tmp/.minio:/root/.minio
   ```

   



