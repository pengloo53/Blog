---
title: 博客自动化部署
date: 2022-11-04
tags: [博客,git]
---

这篇文章拖了一年，还是去年折腾博客的时候，在了解相关技术过程中，记录下来的。目的是为了解决博客自动化部署的问题，这次继续一鼓作气，再填一坑。

<!-- more -->

博客使用的是 Jekyll 搭建的，经过[今年的折腾](/2022/10/20/blog.html)，将服务从 GitHub 转移到了自己的服务器上，与之前相比，在更新博客的时候，有个体验上的差异。

通过 GitHub Pages，只需要将本地博客 push 到 GitHub 仓库上，GitHub 就帮助完成了博客的自动部署，而使用自己的服务并没有自动部署的这个功能，虽然通过 nginx 做了一层反向代理，但也只是解决了访问稳定性的问题，并没有解决自动部署的问题。每次在本地更新了博客之后，先要 push 到 GitHub 上（起到类似云盘的功能），然后再到服务器上，pull 一下博客内容才可以。

这篇文章便是要解决这多一步 pull 操作的问题，方案很简单，主要有两步操作：

1. 能够将本地项目 push 到自己的服务器上，不用 GitHub 做中转，这便需要一个 Git 服务器；
2. Git 服务器在接收到博客更新后，自动完成部署，类似 GitHub Pages 的功能，这便需要利用 Git 的 hook 机制；

## Git 服务器

难不成要在自己的服务器上搭建一个开源的 GitHub？理论上是可行的，实际上是没有必要的。毕竟也不需要提供什么 Web 服务，自己自己使用而已，一个 Git 裸库，便足以了。

### Git 裸库

Git 裸库便是服务端的中心仓库，裸仓库一般情况下是作为远端的中心仓库而存在的，它不包含**工作区**，不能在这个目录下执行 Git 相关命令。其他非裸仓库可以 push 代码到裸仓库，可以从裸仓库 pull 代码到本地。

### 服务器操作

这里快速过一下命令了，原理便不多解释。

```bash
# 新建 git 用户
sudo useradd git
# 进入 git 用户目录
cd /home/git
# 创建裸库
git init --bare blog.git
# 修改用户目录所属
chown -R git:git /home/git
```

两点建议：

1. 建议创建单独的 git 用户来操作，repo 的用户权限改为 git
2. 本地仓库想要顺利（无需输入密码）push 到服务端的 git 仓库，建议配置[SSH 密钥登录 - SSH 教程 - 网道](https://wangdoc.com/ssh/key)

### 本地操作

默认你已经有了一个本地仓库。

```bash
# 关联远程仓库，xxx 起个名字，ip_address 服务器 IP 地址
git remote add xxx git@ip_address:blog.git
# 提交到远程，自动将当前分支推送到了服务器的中心库
git push xxx
```

到这里，方案的第一步：将本地项目推送到服务器就完成了，不需要 GitHub 或其他 Git 服务商作为中转，项目数据完全在自己手中。

接下来，便是自动部署了。

## 自动部署

实现类似于 GitHub Pages 服务的功能，自动部署静态页面，并提供 Web 访问。

Web 服务自然还是用 `nginx`，现在要解决的问题便是 git 服务端仓库，在收到 commit 的时候，自动运行 `jekyll build` 命令，生成静态页面，并放到 `nginx` 的目录下，这就要用到 Git 的钩子功能。

### Git 钩子

Git 提供在一些事件的前后事件节点，自动运行脚本的能力，这便是钩子。上面例子中，我们便是要在服务端更新了项目代码之后，自动跑一段脚本，生成页面。

详细了解，建议看一下官方文档，尽量看英文的，中文翻译有缺失。

- [Git - githooks Documentation](https://git-scm.com/docs/githooks)
- [Git - Git 钩子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)

这里我们要用到的钩子是 `post-update`，该文件在 Git 项目的 hooks 目录下。

### 自动部署

这里主要是服务端的操作，可分为以下几个步骤：

**（1）初始化工作目录**

Git 裸库只是提供了一个可以接收 push 的远程仓库，它并没有工作目录（不会显示项目代码），所以先初始化一个工作目录，可以从本地仓库直接拉取。

```bash
git clone /home/git/blog.git
```

**（2）编写 `post-update` 钩子**

钩子文件在 `/home/git/blog.git/hooks/post-update`，这个目录下有一些钩子的示例代码，可以复制一份开始编辑。

```bash
# 进入目录
cd /home/git/blog.git/hooks
# 复制示例代码
cp post-update.sample post-update
# 开始编辑钩子代码
vim post-update
```

写入代码如下：

```shell
#! /bin/bash
unset GIT_DIR
GIT_CLONE=/home/Git/pengloo53
cd $GIT_CLONE
git pull
exit
```

到这里，每次将代码从本地推送到服务端的时候，服务端的 Git 裸库便会触发钩子，自动执行上述脚本，更新 git 工作仓库目录。

然后结合之前 [Docker 那篇文章关于线上部署的介绍](/2022/10/12/docker-introduction.html#%E7%BA%BF%E4%B8%8A%E9%83%A8%E7%BD%B2)，便可自动完成 Jekyll 站点的部署。

>  写到这里，我突然发现，整个过程在服务端除了安装 git 和 docker，再无需配置任何环境，使用 Docker 真的太方便了。

更多阅读：[Git 进阶学习笔记 · lupeng · blog](/2015/12/14/Git_advance.html)

## 参考

- [部署方法 - Jekyll • 简单静态博客网站生成器](http://jekyllcn.com/docs/deployment-methods/)
- [Git钩子 · Bing's Blog](https://azmddy.github.io/article/%E5%85%B6%E5%AE%83/git%E9%92%A9%E5%AD%90.html)
- [自建Github Pages · Bing's Blog](https://azmddy.github.io/article/%E5%85%B6%E5%AE%83/%E8%87%AA%E5%BB%BAgithubpages.html)
- [linux搭建git服务器_fenlin88l的博客-CSDN博客_linux安装git服务器](https://blog.csdn.net/fenlin88l/article/details/89151075?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link)
- [git利用post-receive自动化部署_weixin_33816300的博客-CSDN博客](https://blog.csdn.net/weixin_33816300/article/details/89009334)
