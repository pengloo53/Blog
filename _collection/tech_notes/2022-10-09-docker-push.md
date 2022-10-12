---
title: Docker 发布镜像
date: 2022-10-09
tags: [docker]
---

这篇文章简单收个尾，入门学习就到这里了。

### 项目地址

自己用来学习测试用的，仅做参考：[Github：pengloo53/docker-jekyll](https://github.com/pengloo53/docker-jekyll)

### 发布镜像

三条命令

```bash
# 登录
docker login
# 标记
docker image tag docker-jekyll:0.0.3 pengloo53/docker-jekyll:0.0.3
# 推送
docker image push pengloo53/docker-jekyll:0.0.3
```

![](/image/docker/image-20221009171903171.png)

同时发布了 0.0.2 和 0.0.3 的版本，0.0.2 可以直接预览本地博客，0.0.3 配合 `nginx` 协同使用。

Docker Hub 的项目地址：[pengloo53/docker-jekyll](https://hub.docker.com/r/pengloo53/docker-jekyll/tags)

具体的使用方法，笔记中已经说明，不再阐述。