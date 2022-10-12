---
title: Docker 常用命令
date: 2022-05-03
tags: [docker,微服务]
---

常用命令统一放在这里，便于更新和查询。

### 镜像 image

```bash
# 列出本机的所有 image 文件。
docker image ls

# 删除 image 文件
docker image rm [imageName]

# 从仓库中拉取 image，不带 tag，便默认 latest
docker image pull [imageName]:[tag]

# 创建 image，基于 Dockerfile 文件
# -t 指定 image 文件名称，可指定标签，最后指定 Dockerfile 的路径，--no-cache 清空缓存，不清空会出现 <none> 的 image
docker image build -t xxx .
docker image build -t xxx:0.0.1 .
docker image build --no-cache -t xxx:0.0.1 .
# 带用户名信息，为了发布到仓库
docker image build -t [username]/[imageName]:[tag] .

# 登录仓库，在命令行发布
docker login
# 本地 image 标注用户信息
docker image tag [imageName] [username]/[imageName]:[tag]
# 发布 image 到仓库
docker image push [username]/[imageName]:[tag]
```

注意：发布镜像需要在 [hub.docker.com](https://hub.docker.com/) 上注册账号，另外，尽量使用别人制作的镜像，多从[官方仓库](https://hub.docker.com/)看看。

### 容器 container

```bash
# 列出本机正在运行的容器
docker container ls
# 列出本机所有容器，包括终止运行的容器
docker container ls --all

# 删除容器文件
docker container rm [containerID]

# 终止运行容器文件，stop 软终止，kill 强制终止
docker container stop [containID]
docker container kill [containID]

# 启动容器，不会新建容器文件
docker container start [containID]

# 终端日志，Shell 的输出
docker container logs

# 进入正在运行的容器
docker container exec -it [containerID] /bin/bash

# 拷贝容器的文件到本地
docker container cp [containID]:[/path/to/file] .
```

### 容器运行 run

```bash
# 从 image 生成一个运行的实例
docker run hello-world
# 自定义容器的名称，--name 参数指定容器名称
docker run --name helloworld hello-world
# 后台运行，不占用终端
docker run -d hello-world

# 命令行体验 Ubuntu 系统，运行服务，-it 映射 Shell
docker run -it ubuntu bash
# 映射本地端口，-p 本地 8000 映射容器 3000 端口
docker run -p 8000:3000 -it koa-demo /bin/bash
# 端口随机映射 -P(大写p)
docker run -P -it koa-demo /bin/bash
# 容器终止运行后，自动删除容器文件
docker run --rm -p 8000:3000 -it koa-demo /bin/bash
```

注意：容器停止，并不代表容器文件删除，想要停止便自动删除文件，运行时需要加参数 `--rm`。