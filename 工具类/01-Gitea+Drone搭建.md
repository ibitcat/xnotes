- Gitea 版本：1.18.1
- Drone 版本：2
- WSL2 ip 地址：192.168.0.88

## 安装 Gitea
使用 docker 方式安装，且使用 docker compose 编排。docker-compose.yml 配置文件如下：
```
version: "3"

services:
  #gitea
  gitea:
    image: gitea/gitea:1.18.1
    container_name: gitea
    #network_mode: "host"
    restart: always
    volumes:
      - ./data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - ROOT_URL="http://192.168.0.88:3000"
      #- SSH_PORT=2222
      #- HTTP_PORT=3000
    ports:
      - "3000:3000"
      - "2222:22"
```
注意需要在配置文件 `data/gitea/conf/app.ini` 增加以下内容：
```
[webhook]
ALLOWED_HOST_LIST = 192.168.0.88
```
否则，gitea 的 webhook 会触发失败。

启动后的设置参考 drone 的[官方文档](https://docs.drone.io/server/provider/gitea/)。

## 安装 drone server
为Drone的管理提供了Web页面，用于管理从Git上获取的仓库中的流水线任务。

docker-compose.yml 配置文件如下：
```
version: "3"

services:
  # drone server
  drone-server:
    restart: always
    image: drone/drone:2
      #network_mode: "host"
    container_name: drone-server
    volumes:
      - ./data:/var/lib/drone/
    environment:
      - DRONE_GITEA_SERVER=http://192.168.0.88:3000 # 支持http, https
      - DRONE_GITEA_CLIENT_ID=6a185fc7-a815-4cf1-861b-4cc81754223c
      - DRONE_GITEA_CLIENT_SECRET=gto_mub54n56bfhyiphgyfxsfgvs62dk6ol74ulskthvpfxrmpku6lya
      - DRONE_RPC_SECRET=baca0a73e531407124948ca82e3912c8
      - DRONE_SERVER_HOST=192.168.0.88:8090
      - DRONE_SERVER_PROTO=http # 支持http, https
      - DRONE_GIT_ALWAYS_AUTH=false
      - DRONE_DEBUG=true
    ports:
      - "8090:80"
```
- DRONE_GITEA_CLIENT_ID，对应 gitea 的 OAuth2 客户端id
- DRONE_GITEA_CLIENT_SECRET，对应 gitea 的 OAuth2 客户端密钥
- DRONE_RPC_SECRET，是 drone 和 runner 之间rpc调用的密钥，可以用 openssl 命令生成

进入 drone web 页面，同步git仓库并激活仓库。

## 安装 drone runner
一个单独的守护进程，会轮询Server，获取需要执行的流水线任务，之后执行。

docker-compose.yml 配置文件如下：
```
version: "3"

services:
  # drone runner
  drone-runner:
    restart: always
    image: drone/drone-runner-docker:1
    #network_mode: "host"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_RPC_PROTO=http # 支持http, https
      - DRONE_RPC_HOST=192.168.0.88:8090
      - DRONE_RPC_SECRET=baca0a73e531407124948ca82e3912c8
      - DRONE_RUNNER_NAME=drone-runner-docker
      - DRONE_RUNNER_CAPACITY=2
    ports:
      - "3002:3000"
```
- DRONE_RPC_SECRET，对应上面 drone server 中的 rpc 密钥

## 测试
在 gitea 上创建一个仓库并加入 `.drone.yml` 配置文件，内容如下：
```
kind: pipeline
type: docker
name: mydrone_test

clone:
  disable: true

steps:
  - name: 编译步骤
    image: alpine
    commands:
      - echo hello drone
```
如果一切正常，则可以在 drone web 管理页面中看到自动触发了构建。构建日志大致如下：
```
latest: Pulling from library/alpine
Digest: sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a
Status: Downloaded newer image for alpine:latest
+ echo hello drone
hello drone
```

## 参考
- [自动化部署之旅](https://blog.51cto.com/palxp/5712475)
- [轻量级CI/CD自动构建平台Gitea+Drone保姆级实践教程](https://blog.csdn.net/ywch520/article/details/124782654)