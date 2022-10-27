---
title: 搭建博客的一些技术记录
date: 2021-11-04
tags: [博客,技术]
---

这篇关于 Git 的笔记，拖更了快一年了，还是去年折腾博客的时候，在学习的过程中记录下来的。学习的目的是为了解决快速更新博客的问题，这次继续一鼓作气，再填一坑。

<!-- more -->

博客使用的是 Jekyll 搭建的，经过[这次的折腾](/2022/10/20/blog.html)，将服务从 GitHub 转移到了自己的服务器上，与之前相比，在更新博客的时候，有个体验上的差异：通过 GitHub Pages，只需要将本地博客 push 到 GitHub 上，GitHub 就帮助完成了博客的自动部署，而使用自己的服务并没有自动部署的这个功能，虽然通过 nginx 做了一层反向代理，但也只是解决了访问稳定性的问题，并没有解决自动部署的问题。每次在本地更新了博客之后，先要 push 到 GitHub 上（起到类似云盘的功能），然后再到服务器上，pull 一下博客内容才可以。

这篇文章便是要解决这多一步 pull 操作的问题，方案很简单，主要有两步操作：

1. 能够将本地项目 git push 到服务器上，去掉使用 GitHub 做中转，需要一个 Git 服务器；
2. 服务器在接收到博客更新后，自动完成部署，需要利用 Git 的 hook 机制；

## Git 服务器

本质上是要实现类似 GitHub 提供的服务，可以将本地的项目通过 git push 到自己的服务器上，那便需要一个 Git 服务器。

难不成要在自己的服务器上搭建一个开源的 GitHub？理论上是可行的，实际上是没有必要的。毕竟也不需要提供什么 Web 服务，也不用提供给别人使用等等。

一个 Git 裸库，便足以了。

### Git 裸库

服务端的中心仓库，裸仓库一般情况下是作为远端的中心仓库而存在的，它不包含**工作区**，不能在这个目录下执行 Git 命令。其他非裸仓库可以 push 代码到裸仓库，可以从裸仓库 pull 代码到本地。

**1. 创建裸库**

生成一个裸仓库。

```bash
cd /home/repo
git init --bare xxx.git
```

以上命令会在 `/home/repo` 目录下生成 `xxx.git` 的目录，即为裸仓库。

**2. 切换裸库的分支**

在裸库中执行 `git checkout <branch-name>` 会报错如下：

```bash
fatal: this operation must be run in a work tree
```

因为裸库是不存在工作区的，可使用命令 `git symbolic-ref HEAD refs/heads/<branch-name>` 替代。

**3. 与其他远程仓库镜像**

````bash
git push --mirror http://github.com/pengloo53/xxx
````

### 服务器配置

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

### 本地操作

默认已经有了一个本地仓库。

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

（1）从本地拉取工作目录

```bash
# 直接从本地路径拉取
git clone /home/git/blog.git

# 如果之前已经使用了 github 拉取，那么，更改项目远程地址为本地
git remote set-url origin /home/git/blog.git
```

（2）编写 `post-update` 钩子

```shell
#! 
GIT_REPO=/home/repo/xxx.git
TMP_GIT_CLONE=/home/git/tmp/blog-site
PUBLIC_WWW=/home/www/xxx
```



然后编辑 `xxx.git` 目录下的 `hooks/post-receive`  文件，该文件的作用：在提交代码之后，`git` 用户会自动执行里面的脚本，实现自动化部署。

```bash
git --work-tree=/home/www/xxx --git-dir=/home/repo/xxx.git checkout -f
```

#### 部署静态页面

```bash
# 指定我的代码检出目录（服务访问目录）
DIR=/home/www/xxx
git --work-tree=${DIR} clean -fd
# 直接强制检出
git --work-tree=${DIR} checkout --force
```

```bash
#!/bin/sh
unset GIT_DIR  #很关键
NowPath=`pwd`
DeployPath="/usr/local/nginx/html/student" #存放项目的文件夹位置
cd $DeployPath
git add . -A && git stash
git pull origin master
composer install --ignore-platform-reqs
cd $NowPath
echo "同步完成"
exit 0
```

#### 部署 Jekyll 服务

```bash
#! /bin/bash
unset GIT_DIR
GIT_REPO=/home/repo/xxx.git
TMP_GIT_CLONE=/home/git/tmp/blog-site
PUBLIC_WWW=/home/www/xxx
git clone $GIT_REPO $TMP_GIT_CLONE
cd $TMP_GIT_CLONE
bundle install
bundle exec jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW -q
rm -rf $TMP_GIT_CLONE
exit
```



## 附录：Git 命令

#### 分支查看与创建

```bash
git branch #查看本地分支
git branch -r #查看远程分支
git branch -a #查看所有分支
git branch xxx #创建分支 xxx

git checkout xxx #切换到分支 xxx
git checkout -b xxx #创建分支 xxx，并切换到 xxx 上（基于当前 HEAD 指向的分支，创建新分支）

git checkout -b new-branch existing-branch  #基于 existing-branch 创建 new-branch，new-branch 和 existing-branch 可以是远程分支，例如：origin/xxx

git fetch --all #拉取所有远程分支
```

#### 分支删除与更改

```bash
git branch -d xxx
git branch -D xxx
git branch -m <new-branch-name> 将当前分支重命名
```

#### 删除远程分支

```bash
git push origin --delete branch_name
git push origin :branch_name
```







## 注意事项

1. git repo 建议创建单独的 git 用户来操作，repo 的用户权限改为 git
2. 本地仓库想要顺利（无需输入密码） push 到服务端的 git 仓库，建议[配置 ssh 的密钥登陆]()





## 参考

- [hexo+阿里云搭建博客网站](https://qianguyihao.com/post/2020-09-19-hexo-aliyun-blog/)
- [为Github page绑定自定义域名并实现https访问](https://blog.csdn.net/yucicheung/article/details/79560027)
- [为GitHub Pages自定义域名并添加SSL-开启https强制](https://javef.github.io/2018/04/%E4%B8%BAGitHub-Pages%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E5%B9%B6%E6%B7%BB%E5%8A%A0SSL-%E5%BC%80%E5%90%AFHTTPS%E5%BC%BA%E5%88%B6/#:~:text=%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5%E4%B8%8B%E4%BD%BF%E7%94%A8GitHub%20Pages%E7%9A%84%E7%BB%99%E5%AE%9A%E5%9F%9F%E5%90%8D%E5%88%99%E6%94%AF%E6%8C%81http%E5%92%8Chttps%E4%B8%A4%E7%A7%8D%E5%8D%8F%E8%AE%AE%EF%BC%8C%E4%BD%86%E6%98%AF%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E7%9A%84%E8%AF%9D%EF%BC%8C%E5%88%99%E5%8F%AA%E8%83%BD%E9%80%9A%E8%BF%87%20http%3A%2F%2F%20%E8%AE%BF%E9%97%AE%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E8%AF%B4%E6%88%91%E4%BB%AC%E5%9C%A8%20Github%E4%B8%8A%E6%90%AD%E5%BB%BA%20Hexo,%E6%88%96Jekyll%20%E4%B8%BB%E9%A2%98%E5%8D%9A%E5%AE%A2%20%E5%90%8E%EF%BC%8C%E9%80%9A%E8%BF%87%20CNAME%20%E7%BB%91%E5%AE%9A%E4%B8%AA%E4%BA%BA%E5%9F%9F%E5%90%8D%E5%90%8E%EF%BC%8C%E6%88%91%E4%BB%AC%E5%8F%AA%E8%83%BD%E9%80%9A%E8%BF%87%20http%3A%2F%2F%20%E5%9F%9F%E5%90%8D%E6%9D%A5%E8%AE%BF%E9%97%AE%E3%80%82)
- [SSL 证书 一键 HTTPS - 证书安装 - 文档中心 - 腾讯云](https://cloud.tencent.com/document/product/400/58062)
- [Git钩子 · Bing's Blog](https://azmddy.github.io/article/%E5%85%B6%E5%AE%83/git%E9%92%A9%E5%AD%90.html)
- [自建Github Pages · Bing's Blog](https://azmddy.github.io/article/%E5%85%B6%E5%AE%83/%E8%87%AA%E5%BB%BAgithubpages.html)
- [linux搭建git服务器_fenlin88l的博客-CSDN博客_linux安装git服务器](https://blog.csdn.net/fenlin88l/article/details/89151075?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link)
- [git利用post-receive自动化部署_weixin_33816300的博客-CSDN博客](https://blog.csdn.net/weixin_33816300/article/details/89009334)





> zsh: /usr/bin/yum: bad interpreter: /usr/bin/python: 没有那个文件或目录

yum 没有配置正确，多半是安装 python 环境导致



[使用yum命令的时候报，-bash: /usr/bin/yum: /usr/bin/python: bad interpreter: 没有那个文件或目录_anning_88的专栏-CSDN博客](https://blog.csdn.net/anning_88/article/details/75735757)



[CentOS 7 使用 rvm 安装 ruby 搭建 jekyll 环境 - Zhanming's blog](https://qizhanming.com/blog/2017/05/31/install-rvm-and-ruby-buid-jeklly-env-on-centos-7)



版本依赖关系查看

[jekyll | RubyGems.org | Ruby 社群 Gem 套件管理平台](https://rubygems.org/gems/jekyll/versions/4.2.1)



