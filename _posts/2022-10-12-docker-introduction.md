---
title: Docker Introduction
date: 2022-10-12
tags: docker
pin: true
---

2022 年了，才开始去学习 Docker，也是没谁了。当然，也不是闲着没事，最初的目的是为了解决在 Windows 上搭建 Jekyll 服务的问题，便于在上班时间摸鱼发文章，嗯，最初的动机就是这么的单纯。

在学习 Docker 之前，其实也尝试过直接在 Windows 下部署 Ruby 环境，但总是因为一些原因而失败，Ruby 似乎对 Windows 非常不友好，后来索性不折腾了，学点 Docker 吧。

本文分为以下 6 个部分：

- 开始：简单了解，大致走一遍使用流程
- 常用命令：汇总记录一些常用的命令，便于查阅
- 制作镜像：进一步了解 `Dockerfile` 文件
- 搭建服务：完成 `Jekyll` 本地预览服务的搭建
- Compose：联合 `Nginx` 服务，初步了解 Docker Compose
- 结尾：发布镜像，正式环境验证

<!-- more -->

## 开始

### 介绍

直接看下大佬文章，[Docker 入门教程](https://ruanyifeng.com/blog/2018/02/docker-tutorial.html)

### 安装

Windows 和 Mac 的安装就不多说了。

- Windows 下载地址：[Docker Desktop for Mac and Windows](https://www.docker.com/products/docker-desktop)
- Mac 下载地址：[Docker Desktop - Docker](https://www.docker.com/products/docker-desktop/)

CentOS 8 安装 Docker，主要参考此文，[CentOS8 安装 Docker-阿里云开发者社区](https://developer.aliyun.com/article/753261?accounttraceid=778f786f99dc4a1689b886735dbdb33bfmei)，摘录一下主要内容如下：

#### 1. 卸载老版本

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 2. 安装docker 基础包

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

#### 3. 设置稳定仓库

```bash
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### 4. 安装Docker Engine - Community

- 安装最新版本(latest)

```
yum install docker-ce docker-ce-cli containerd.io
```

- 安装指定版本

```
yum list docker-ce --showduplicates | sort -r  #查看版本
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io #安装指定版本
```

#### 5. 启动与测试

```
sudo systemctl start docker  # 启动docker
docker run hello-world  #测试
```

运行后会显示下图，说明docker安装成功。

![](/image/docker/image-20220419134523023.png)

> PS. 安装完 Docker 后，建议先不要使用其图形客户端，学会基础的命令之后，再来看客户端提供的功能，你会发现客户端只剩下好看了。

### 使用

#### 简单应用

一般的使用流程是：直接用别人制作好的镜像，然后启动容器。例如下面这条命令：

```bash
docker run --rm -p 8000:80 nginx:latest
```

这条命令，先会在本地找一个名为 `nginx`，版本为 `latest` 的镜像，如果没有，就会去官方仓库中拉取，然后 run 一个容器。

![](/image/docker/image-20221009143853564.png)

就这样，一个 nginx 的服务就起来了，在浏览器中访问 `http://localhost:8000`，便可看到 nginx 服务的首页。

![](/image/docker/image-20221009144008064.png)

而实际使用场景比这个要复杂，通常要从定制一个镜像开始。

#### 制作镜像

制作镜像需要一个叫 `Dockerfile` 的配置文件，一个纯文本文件，用来生成 image，Docker 根据该文件生成二进制的 image。

找一个目录，创建 `Dockerfile` 文件，里面编写如下几行代码：

```bash
FROM nginx
RUN echo "Hello Nginx with Docker" > /usr/share/nginx/html/index.html
```

这是一个非常简单的示例，就两行代码：

- `FROM nginx` 表示该 image 将继承自官方 nginx 镜像，没有指定版本，那么就是默认继承最新 latest 版本
- `RUN` 命令表示，在创建 image 之前，执行的命令，也就是说，这个创建好的镜像将会有一个 index.html 的文件

定义好之后，开始创建镜像，在 Dockerfile 目录下执行如下命令：

```bash
docker image build -t nginx-demo:0.1 .
```

![](/image/docker/image-20220429095617436.png)

一个名称叫 nginx-demo 的 image 就创建成功了，然后就可以基于这个 image 启动一个容器。

#### 启动容器

启动容器的命令如下：

```
docker run -p 8080:80 --name nginx-demo nginx-demo:0.1
```

以上命令省略了 `container` 这个词，其实意思是一样的。

![](/image/docker/image-20220429154650142.png)

如上命令行显示，表示启动成功了，在浏览器中访问：`http://localhost:8080` 即可在网页上看到 「Hello Nginx with Docker」这句话。

#### 映射本地目录

上面例子是在创建 image 的时候，写入了一个 index.html 文件，启动容器之后，就可以访问到了。

如果本地有一个静态站点，如何让它在 Docker 的容器中跑起来呢？其实，很简单，把本地的静态站点目录映射到容器中就可以了。

重新启动一个容器，命令如下：

```
docker run --rm -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx-demo:0.1
```

上面多了个 `-v` 的命令，也可以写成 `--volume`，它的作用是映射目录，将本地目录映射到容器中，后面跟着的参数便是对应的目录，用 `:` 相隔。

- `$(pwd)` 表示的是本地的目录，这里我们放一个静态站点，或者放一个 html 文件也可以；
- `/usr/share/nginx/html` 即为 Nginx 容器默认的站点目录，具体可以进入容器内部，从 nginx 的配置文件中查看到。

那么，如何能够进入容器内部看看呢？

#### 映射命令行

启动一个容器，使用如下命令：

```bash
docker run --rm -it nginx-demo:0.1 /bin/bash
```

- `it` 的作用是映射 Shell，这样就可以在本地运行容器的命令了

通过如下一系列命令，就可以查看到 Nginx 的配置文件。

```bash
cd /etc/nginx/conf.d
cat default.conf
```

![](/image/docker/image-20220430224633836.png)

在这个命令行下，我们还能够查看到这个 nginx 镜像是基于哪个 Linux 系统制作的。运行这个命令试试：`cat /etc/issue`。

#### 发布镜像

发布这个环节就省了，这个程度的镜像根本就不能称之为自制的镜像。这个示例只是为了熟悉一下命令而已。

通过这个示例也大概了解了一个容器的生成和基本的使用，如果想要深入了解如何制作一个镜像，除了要熟悉 [Dockerfile](https://docs.docker.com/engine/reference/builder/) 的一些基本语法，还要对 Linux 命令行有一定的了解。

## 常用命令

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

## 制作镜像

下面将通过实操，进一步了解制作镜像的一些知识点。

<!-- more -->

### Copy  一个镜像

我们知道制作一个镜像，需要编写 Dockerfile 配置文件。但是，我并没有上来就开始学习语法，编辑 Dockerfile，而是在仓库中找了一个 Jekyll 的镜像，先看看别人怎么写的。

![](/image/docker/image-20220505134048977.png)

上面镜像的地址是 [BretFisher/jekyll-serve](https://github.com/BretFisher/jekyll-serve)，关于 Jekyll 的镜像比较多，但是能正常跑起来却没有多少，因为年代都太久远了。

在 Jekyll blog 的根目录下，使用下面的命令就可以启动容器了

```bash
docker run --name jekyll-test --rm -p 4000:4000 -v $(pwd):/site bretfisher/jekyll-serve
```

上面这条命令的参数解释如下：

- `--name` 给容器自定义一个名称
- `--rm` 表示在容器停止后，自动删除容器文件
- `-p` 端口映射，本地:容器
- `-v` 目录映射，本地:容器，`$(pwd)` 表示本地运行 `pwd` 命令的结果，也就是代表当前目录，这里的 `$(pwd):/site`  指的是映射本地当前目录到容器里的 /site 目录下

Windows 下可能会出现 Access Denied 的提示，本地目录建议使用 `/C/Users/Administrator(用户名)` 下的目录根路径来替代 `$(pwd)`，或者通过图形界面来启动容器。

![](/image/docker/image-20220427202432004.png)

![](/image/docker/image-20220428104609595.png)

图形界面的选择项很简单，就不多解释了，基本上就是对应上面介绍的几个参数值。

到这里，一个 jekyll 的环境就跑起来了，可以满足在本地预览博客的需求了。但学习并未止步。

### Build  一个镜像

打开别人的项目，看看 Dockerfile 是怎么写的。如下代码：

```bash
FROM ruby:2-alpine as jekyll

RUN apk add --no-cache build-base gcc bash cmake git gcompat

# install both bundler 1.x and 2.x incase you're running
# old gem files
# https://bundler.io/guides/bundler_2_upgrade.html#faq
RUN gem install bundler -v "~>1.0" && gem install bundler jekyll

EXPOSE 4000

WORKDIR /site

ENTRYPOINT [ "jekyll" ]

CMD [ "--help" ]


FROM jekyll as jekyll-serve

COPY docker-entrypoint.sh /usr/local/bin/

# on every container start, check if Gemfile exists and warn if it's missing
ENTRYPOINT [ "docker-entrypoint.sh" ]

CMD [ "bundle", "exec", "jekyll", "serve", "--trace", "--force_polling", "-H", "0.0.0.0", "-P", "4000" ]
```

简单解释下上述代码的含义：

- `FROM ruby:2-alpine as jekyll`：该 image 文件继承官方的 ruby 镜像，冒号后面是版本标签
- `RUN apk add --no-cache build-base gcc bash cmake git gcompat` 使用 alpine 的 apk 包管理器工具，在该镜像里，安装一些基础软件工具
- `RUN gem install bundler -v "~>1.0" && gem install bundler jekyll` 继续安装jekyll 服务需要的一些软件工具，通过 ruby 下的 gem 安装
- `EXPOSE 4000` 对外暴露 4000 端口
- `WORKDIR /site`：指定工作目录为`/app`。
- `COPY . /app`：将当前目录下的所有文件（除了`.dockerignore`排除的路径），都拷贝进入 image 文件的`/app`目录
- `ENTRYPOINT [ "jekyll" ]` 镜像的默认入口命令，需要与下面那条 CMD 命令结合来看
- `CMD [ "--help" ]` --help 作为 jekyll 的参数执行，也就是安装完 jekyll 之后，会执行`jekyll --help` 的条命令

到这里，其实 jekyll 的镜像已经制作完毕，咱们先不看最后的 4 行代码，把代码 copy 出来，自己先 build 下试试。

新建一个项目，新建 Dockerfile 文件，贴入代码。然后，打开命令行，输入命令：

```bash
docker image build -t docker-jekyll:0.0.1 .
```

创建一个名为 docker-jekyll，版本为 0.0.1 的镜像。

加载完之后，会看到电脑里多了一个镜像，执行 `docker image ls` 或者看图形界面。

![](/image/docker/image-20220928152429475.png)

执行命令 `docker run --rm docker-jekyll:0.0.1`，跑一个容器试试，输出如下，说明该镜像一切正常。

![image-20220928153222586](/image/docker/image-20220928153222586.png)

可以发现，控制台打印信息，正好是命令 `jekyll --help` 的输出。

接着来，我们使用下面这条命令，整点有用的。

```bash
docker run --rm -v /C/Users/Administrator/Git/docker-jekyll/site:/site docker-jekyll:0.0.1 new .
```

你会发现，在你的项目目录下，使用 `jekyll new .` 命令，生成了一个 Jekyll 的博客模板。

这个博客模板是在刚刚创建的 Jekyll 镜像里生成的，并映射到了本地目录上，也就是说，**在本地没有搭建 Jekyll 服务的前提下，使用 Jekyll 命令在本地生成了一个博客目录。**

## 搭建服务

通过上面的介绍，已经大致了解，镜像怎么来的、怎么制作以及如何使用，并且，通过镜像里的 Jekyll 服务，在本地生成了一个 Jekyll 博客模板。

博客倒是生成了，怎么把博客服务运行起来，并且能够在主机访问到呢？接下来便要完成这件事情。

首先，我们需要了解的是，Jekyll 博客要在本地跑起来（也就是开启服务模式），需要用到一个命令：`jekyll serve`，并且还得知道，运行这个命令的时候，会在本地安装一些依赖包。

而上次制作的那个镜像，显然不满足这样的条件，可以运行下面命令试一下。

```bash
docker run --rm -v /C/Users/Administrator/Git/docker-jekyll/site:/site docker-jekyll:0.0.1 serve
```

报错如下：

![](/image/docker/image-20220929155916101.png)

这个报错大致意思是，`jekyll serve` 服务跑不起来，因为缺少很多的依赖包。

而依赖包的关系是在项目目录的 `Gemfile` 文件里写着的，打开生成的那个博客目录，可以看到这个文件。

![](/image/docker/image-20220929160439274.png)

也就是说，我们首先需要把这个项目目录映射到镜像中，然后，在镜像里运行安装依赖包的命令 `bundle install`，等依赖包安装完成，最后运行 `jekyll serve` 的命令，才能将 Jekyll 服务跑起来。

这里涉及一个问题，那就是：如何在镜像创建完之后，执行多条命令？我们知道，执行命令使用的指令是 `ENTRYPOINT` 和 `CMD` 语句。而这个语句都是不能写多个的，并且，两者之间也是有一些关联关系的。

具体可以看下这篇文章[《Dockerfile 中的 CMD 与 ENTRYPOINT》 ](https://www.cnblogs.com/sparkdev/p/8461576.html)，其中有详细的介绍两者的区别和关系。

要执行多条命令，其实有多种方案，这里我选择最简单的方法，就是把两条命令拼一块执行。制作镜像的代码如下：

```bash
FROM ruby:2-alpine as jekyll

RUN apk add --no-cache build-base gcc bash cmake git gcompat

# install both bundler 1.x and 2.x incase you're running
# old gem files
# https://bundler.io/guides/bundler_2_upgrade.html#faq
RUN gem install bundler -v "~>1.0" && gem install bundler jekyll

EXPOSE 4000

WORKDIR /site

ENTRYPOINT sh -c "bundle install && bundle exec jekyll serve --trace -P 4000 -H 0.0.0.0"
```

前面几行命令，不再解释，只看最后一条命令。`ENTRYPOINT` 指令后面跟着一个命令行，使用的是 shell 模式，shell 模式下， `CMD` 指令就没用了，这里也不需要了。

```bash
sh -c "bundle install && bundle exec jekyll serve --trace -P 4000 -H 0.0.0.0"
```

上面这条命令，其实就是执行前面说到的两条命令，`bundle install`是安装依赖，`&&` 后面那个是启动 Jekyll 服务。`sh -c` 指的是将后面的字符串当作命令来执行。

解释完 `Dockerfile` 文件，就可以生成 image 了，这里还是在之前的镜像上，新打了个版本标签。命令如下：

```bash
docker image build -t docker-jekyll:0.0.2 .
```

紧接着，可以跑容器了。命令如下：

```bash
docker run --rm -p 4001:4000 -v /C/Users/Administrator/Git/docker-jekyll/site:/site docker-jekyll:0.0.2
```

这里一定要指定端口号，`-p 4001:4000` 将容器内部 4000 端口映射到外部 4001 端口。

最后，打开浏览器，访问 `http://localhost:4001`，便可以正常预览博客了。

![](/image/docker/image-20220930111857656.png)

到这里，我们的目的就达成了，一个 Jekyll 服务终于搭建了起来，通过它可以预览本地的博客了。

### One more thing

前面说到过，要实现在镜像生成之后执行多条命令，有好几种方案。我们在上篇文章中提到的那个作者，便是采用另外一种更通用的方案：执行脚本。

先将本地的脚本拷贝到容器中，然后，使用 `ENTRYPOINT` 设置为入口命令，便可在镜像生成后，执行脚本内容。

为什么说更通用？因为很多情况可能不是一两句命令就能解决问题的，通过脚本可以添加很多自定的功能。比如：`docker-entrypoint.sh` 就添加校验和提示的功能。

```bash
#!/bin/bash
set -e

if [ ! -f Gemfile ]; then
  echo "NOTE: hmm, I don't see a Gemfile so I don't think there's a jekyll site here"
  echo "Either you didn't mount a volume, or you mounted it incorrectly."
  echo "Be sure you're in your jekyll site root and use something like this to launch"
  echo ""
  echo "docker run -p 4000:4000 -v \$(pwd):/site bretfisher/jekyll-serve"
  echo ""
  echo "NOTE: To create a new site, you can use the sister image bretfisher/jekyll like:"
  echo ""
  echo "docker run -v \$(pwd):/site bretfisher/jekyll new ."
  exit 1
fi

bundle install --retry 5 --jobs 20

exec "$@"
```

前提是要将脚本拷贝到容器中，所以，镜像中要有如下两行代码。

```bash
......

# copy .sh file to container
COPY docker-entrypoint.sh /usr/local/bin/
# on every container start, check if Gemfile exists and warn if it's missing
ENTRYPOINT [ "docker-entrypoint.sh" ]

......
```

## Docker Compose

通过前面的介绍，基本完成了：在本地（Windows）搭建 Jekyll 的环境，预览博客。现在还差最后一步，如何在生产环境中部署博客？

通过 `jekyll serve` 自带的服务，肯定是不行的，性能和稳定性都不足以支撑在正式环境中运行。针对这样的静态博客，正式环境中，通常的方案是使用 `Nginx` 服务做反向代理。

### Nginx 服务

nginx 的镜像就不需要自己制作了，直接使用官方的便可。运行下面命令就可以直接启动容器了。

```bash
docker run -it --rm nginx:latest bash
```

进入容器的命令行，查看 nginx 的默认配置。

```
cd /etc/nginx/conf.d
cat default.conf
```

![](/image/docker/image-20221008140718824.png)

可以查看到 nginx 默认配置的路径是 `/usr/share/nginx/html` ，我们只需要将 Jekyll 生成的静态页面，放到这个目录下就可以了。

重新再起一个容器，这次挂载目录。

```bash
docker run -d -p 8080:80 -v /C/Users/Administrator/Git/docker-jekyll/_site:/usr/share/nginx/html nginx:latest
```

这次访问 `http://localhost:8080` 便可通过 nginx 访问博客首页了。

### Docker  Compose

上面两个容器需要同时作用才可以实现在 nginx 中部署静态博客，且有先后关系，先要通过 Jekyll 的服务生成静态站点，然后，通过 Nginx 容器反向代理。

针对这样多个容器协同作用的场景，Docker 有更方便的手段，那便是 Docker Compose，Docker Compose 需要单独安装，不过针对 Windows 环境来说，随着图形界面的安装，已经一并安装了。

![](/image/docker/image-20221008162557883.png)

Linux 上安装命令如下：

```bash
# 下载 docker-compose，下载地址需要自行确认
curl -L https://github.com/docker/compose/releases/download/v2.11.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
# 授权
chmod +x /usr/local/bin/docker-compose
# 查看版本号，这里是 2.11.2
docker-compose -v
```



既然都通过 nginx 反向代理静态博客了，那么 Jekyll serve 的本地服务其实就不用再启动了。可以重新制作一个 Jekyll 镜像，修改 Dockerfile 代码如下：

```bash
FROM ruby:2-alpine as jekyll

RUN apk add --no-cache build-base gcc bash cmake git gcompat

# install both bundler 1.x and 2.x incase you're running
# old gem files
# https://bundler.io/guides/bundler_2_upgrade.html#faq
RUN gem install bundler -v "~>1.0" && gem install bundler jekyll

# 不用暴露端口了
# EXPOSE 4000 

WORKDIR /site

# 启动命令修改为 jekyll build，生成静态目录即可。
# ENTRYPOINT sh -c "bundle install && bundle exec jekyll serve --trace -P 4000 -H 0.0.0.0" 
ENTRYPOINT sh -c "bundle install && bundle exec jekyll build --watch"
```

附上 Jekyll build 的用法如下，这里我们只需要生成博客的静态站点便可。

```bash
$ jekyll build
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中。

$ jekyll build --destination <destination>
# => 当前文件夹中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --source <source> --destination <destination>
# => 指定源文件夹<source>中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --watch
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中，
#    查看改变，并且自动再生成。
```

然后，在当前目录下，新建 `docker-compose.yml` 文件，代码如下：

```bash
version: '2.10'
services:
  jekyll:
    image: docker-jekyll:0.0.3
    volumes:
      - ./site:/site
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./site/_site:/usr/share/nginx/html
```

这是一段 compose 的配置代码，很容易理解。

- `version` 指定使用的 compose 版本号，直接使用当前安装的便可
- `services` 申明需要管理的所有容器，每个容器单独配置，可以看到，这里配置了两个服务 `jekyll` 和 `nginx`

至于容器下面的配置参数，上面这段比较简单，从字面上便可理解其含义，就不多解释，更多参数了解可参考[官方文档](https://docs.docker.com/compose/compose-file/)。

接下来，便是启动服务，输入命令 `docker compose up` 即可，在图形界面上，看到的是一个组合容器的样子。

![image-20221009102453144](/image/docker/image-20221009102453144.png)

该联合服务的作用是，jekyll 容器监控博客内容，发现变动便自动更新 _site，而 nginx 服务反向代理静态站点，提供 Web 服务。

## 结尾

最后简单收个尾，将项目代码上传到 `Github`，将镜像发布到 `Docker Hub`，最后在正式环境中，验证一下学习成果。

### 项目地址

供学习测试使用：[Github：pengloo53/docker-jekyll](https://github.com/pengloo53/docker-jekyll)

### 发布镜像

三条命令

```shell
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

具体的使用方法，上面已经详细阐述，不再说明。

### 线上部署

首先你得有一个云主机，并且能够登录进去，主机里已经安装了 Git、Docker 等软件。

我这里系统是 CentOS 8.2 ，阿里云的主机（一定要记得设置阿里云的安全规则）。

```shell
[root@xxx ~]# cat /etc/redhat-release
CentOS Linux release 8.2.2004 (Core)
```

安装 Docker，命令如下（上面已经有了，这里再写一遍）：

```shell
# 安装基础依赖
yum install -y yum-utils device-mapper-persistent-data lvm2
# 切换国内镜像
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装主程序
yum install docker-ce docker-ce-cli containerd.io
# 启动服务
sudo systemctl start docker
```

安装 docke-compose，命令如下：

```shell
# 下载 docker-compose，下载地址需要自行确认
curl -L https://github.com/docker/compose/releases/download/v2.11.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
# 授权
chmod +x /usr/local/bin/docker-compose
# 查看版本号，这里是 2.11.2
docker-compose -v
```

将 [pengloo53/docker-jekyll](https://github.com/pengloo53/docker-jekyll) 项目下载下来，命令如下：

```shell
git clone https://github.com/pengloo53/docker-jekyll
```

修改项目里的 `docker-compose.yml` 文件代码（主要是修改了 `version` 的值和 `nginx` 映射的端口号）：

```yaml
version: '2.11.2'
services:
  jekyll:
    image: pengloo53/docker-jekyll:0.0.3
    volumes:
      - ./site:/site
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./site/_site:/usr/share/nginx/html
```

回到项目目录，启动服务，命令如下：

```shell
docker-compose -p blog up
```

访问公网 IP 即可，到此结束。