---
title: 拿到服务器要做的事情
date: 2022-03-15
tags: [Linux,运维]
---

### SSH 密钥登陆服务器

#### 生成公钥

在本地终端，输入 `ssh-keygen` 命令后，一路回车就可以了。

Windows 电脑建议提前安装 [Git 客户端](https://gitforwindows.org/)，千万别用 cmd 来操作。

![](2022-03-15-get-server-first-do/image-20211102174256261.png)

然后就可以在用户根目录下找到 `.ssh` 目录了，里面有一个 `id_rsa.pub` 的文件，就是本机的公钥了。

#### 复制公钥

复制的话，用下面这个命令把它拷贝一份到 D 盘根目录上。

```shell
cat ~/.ssh/id_rsa.pub > /d/1.txt
```

复制内容，贴到 `Github-Setting-SSH and GPG keys` 里面，就可以不用密码 push 代码了。

如果把它放到服务器上，就可以不用密码登陆服务器了，怎么把公钥上传到服务器上？很简单，只需要下面这条命令就可以了。

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xxx.xxx.xxx.xxx
```

原理就是在把公钥内容，复制到了服务器上 root 用户 `.ssh` 目录下的  `authorized_keys` 文件里。

好了，可以尝试 `ssh root@xxx.xxx.xxx.xxx` 登陆服务器了。

### 配置网络

如果是在云服务商购买的云服务器，默认你的网络是正常的。

### 下载软件

#### CenOS 8 yum 失败

[完美解决CentOS8 yum安装AppStream报错，更新yum后无法makecache的问题 - 白_胖_子 - 博客园](https://www.cnblogs.com/bpzblog/p/13918199.html)

[centos镜像-centos下载地址-centos安装教程-阿里巴巴开源镜像站](https://developer.aliyun.com/mirror/centos)



#### 安装 Git、Nginx 以及 tmux

很简单了，`yum install -y git nginx tmux`  就可以了。





## 常用命令查询

以下都是针对 CentOS 环境下命令。

### 基本信息

查看 Linux 内核版本

- cat /proc/version
- uname -a

查看 Linux 发行版

- cat /etc/issue

- cat /etc/redhat-release

- lsb_release -a

### 常见错误

#### sudo：command not found

1. 没有安装，`find /etc/sudoers.d`，安装 `yum install sudo`
2. 没有配置路径，找不到对应程序。特征：直接能运行，加上 `sudo` 报错

编辑 `/etc/sudoers`，将命令的路径添加到安全路径中。

![](./../image/2022-03-15-get-server-first-do/image-20221019162132166.png)

### 检查端口

远程机器上检测命令：`telnet ip port`

```bash
systemctl status firewalld    # 查看防火墙的状态
systemctl start firewalld    # 开启防火墙
systemctl stop firewalld    # 关闭防火墙
```

### 卸载软件

```bash
# 如果是 yum 安装的，使用 yum 来卸载
yum search xxx # 查询
yum list # 所有
yum list updates # 可更新列表
yum list installed # 已安装列表
yum list extras # 排除 yum repository 范围外的已安装的软件
yum remove xxx # 卸载
yum clean all # 清除缓存


# 如果是 rpm 安装的，使用 rpm 卸载
rpm -qa | grep xxx # 查询是否安装
rpm -e xxx  # 卸载
rpm -e --nodeps xxx  # 强制卸载
```



- [hexo+阿里云搭建博客网站](https://qianguyihao.com/post/2020-09-19-hexo-aliyun-blog/)
- [为Github page绑定自定义域名并实现https访问](https://blog.csdn.net/yucicheung/article/details/79560027)
- [为GitHub Pages自定义域名并添加SSL-开启https强制](https://javef.github.io/2018/04/%E4%B8%BAGitHub-Pages%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E5%B9%B6%E6%B7%BB%E5%8A%A0SSL-%E5%BC%80%E5%90%AFHTTPS%E5%BC%BA%E5%88%B6/#:~:text=%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5%E4%B8%8B%E4%BD%BF%E7%94%A8GitHub%20Pages%E7%9A%84%E7%BB%99%E5%AE%9A%E5%9F%9F%E5%90%8D%E5%88%99%E6%94%AF%E6%8C%81http%E5%92%8Chttps%E4%B8%A4%E7%A7%8D%E5%8D%8F%E8%AE%AE%EF%BC%8C%E4%BD%86%E6%98%AF%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E7%9A%84%E8%AF%9D%EF%BC%8C%E5%88%99%E5%8F%AA%E8%83%BD%E9%80%9A%E8%BF%87%20http%3A%2F%2F%20%E8%AE%BF%E9%97%AE%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E8%AF%B4%E6%88%91%E4%BB%AC%E5%9C%A8%20Github%E4%B8%8A%E6%90%AD%E5%BB%BA%20Hexo,%E6%88%96Jekyll%20%E4%B8%BB%E9%A2%98%E5%8D%9A%E5%AE%A2%20%E5%90%8E%EF%BC%8C%E9%80%9A%E8%BF%87%20CNAME%20%E7%BB%91%E5%AE%9A%E4%B8%AA%E4%BA%BA%E5%9F%9F%E5%90%8D%E5%90%8E%EF%BC%8C%E6%88%91%E4%BB%AC%E5%8F%AA%E8%83%BD%E9%80%9A%E8%BF%87%20http%3A%2F%2F%20%E5%9F%9F%E5%90%8D%E6%9D%A5%E8%AE%BF%E9%97%AE%E3%80%82)
- [SSL 证书 一键 HTTPS - 证书安装 - 文档中心 - 腾讯云](https://cloud.tencent.com/document/product/400/58062)







[使用yum命令的时候报，-bash: /usr/bin/yum: /usr/bin/python: bad interpreter: 没有那个文件或目录_anning_88的专栏-CSDN博客](https://blog.csdn.net/anning_88/article/details/75735757)

[CentOS 7 使用 rvm 安装 ruby 搭建 jekyll 环境 - Zhanming's blog](https://qizhanming.com/blog/2017/05/31/install-rvm-and-ruby-buid-jeklly-env-on-centos-7)



版本依赖关系查看

[jekyll | RubyGems.org | Ruby 社群 Gem 套件管理平台](https://rubygems.org/gems/jekyll/versions/4.2.1)

