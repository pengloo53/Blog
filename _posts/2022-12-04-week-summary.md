---
title: 居家办公，稀碎的时间 ｜ 周总结
date: 2022-12-04
tags: [周总结,kindle2flomo,side project,nginx,cdn]
---

## 工作生活

### 疫情居家

这周基本就在家里待着里，小区这边再也封控不住了，即便出现了阳，也只是封单元楼了，现在舆论基本一边倒，跟 5 月份那会完全不一样了。

小区虽然不限制外出了，但也无法正常上班，周二的时候去了公司一趟，被告知可以不来了，控制到岗率，周三便通知全体居家办公，公司那边出现阳，楼里要进行环境采样，后面又通知，环境采样结果正常，需要办公的可以来，但不建议来，到岗率要控制在 5%，关于疫情防控，这一周总感觉怪别扭的，有一种感觉就是，所有人都在等上面一声令下，全面放开。

### 居家办公

居家办公，最大的问题是，家里有个小魔童不让你办公。

经过两周的适应，也慢慢找到了自己的节奏，居家的这些日子，每天依旧是平时上班点起床（6 点 15），距离孩子醒来，大概有一个来小时，这一个多小时，可以处理那些需要专注才能做的一些工作，例如：产品规划、需求文档等；然后，白天就跟媳妇轮流看娃，被动的处理一些杂事，接个电话，开个会啥的。

这周工作上的事还格外的多，即便上面这样的安排，也只能是刚刚满足工作需要，至于 side project，几乎没投入啥时间。

## side project

### auth0

周二的时候，去了一趟公司，抽空看了下 auth0 的接口，拿着示例代码运行了下，基本满足我的需求。

后续项目中，用户登陆注册、角色权限管理这一块功能完全不用单独做了，一键接入，非常方便。

同时，还可以配置自己的数据库，针对一些用户数据比较重要的项目，可以配置自己的数据库来保存用户数据。

### Vant UI

看 auth0 接口文档的时候，使用 Vant UI 框架，简单写了个 demo，整体来说，比较顺利，项目后续的移动 UI 框架就选它了。

> Vant 是一个**轻量、可靠的移动端组件库**，于 2017 年开源。目前 Vant 官方提供了[Vue 2 版本](https://vant-contrib.gitee.io/vant/v2)、[Vue 3 版本](https://vant-contrib.gitee.io/vant)和[微信小程序版本](http://vant-contrib.gitee.io/vant-weapp)，并由社区团队维护[React 版本](https://github.com/3lang3/react-vant)和[支付宝小程序版本](https://github.com/ant-move/Vant-Aliapp)。

可以说，在移动 UI 框架里，它是最全的了。

### Kindle2flomo

上周已经将 Kindle2flomo 这个工具部署上线了，这周又做了一些移动端的优化，完全是「产品经理病」又犯了，主要是页面上一些交互和样式的细节调整。

![](/image/2022-12-04-week-summary/0578A1CD-1DE8-4676-9444-2204CA3BE1BD.c2f7eb2f653f4c4aadd3214655e03928.jpg)

同时，把 URL 换了下，现在 Kindle2flomo 的访问地址改成了 [https://90byte.com/kindle2flomo/](https://90byte.com/kindle2flomo/)，首页域名（[https://90byte.com](https://90byte.com)）改成了下面这个页面。

![](/image/2022-12-04-week-summary/E68CB983-43B0-44B1-96A6-8BD538BB5CBF.d0ea9343bab3485cb5185fc0a501ec0c.jpg)

这个页面很久之前便写好了，这次重启 side project 再正式拿出来，作为起始页吧。

### ngxin 配置

如上场景，我需要在一个 server 上部署多个静态站点，nginx 的配置上要做一些调整，最终调整成下面这样：

```nginx
...
location / {
    root /home/www/index
    index index.html;
    tryfiles $uri $uri/ /index.html;
}
location /kindle2flomo {
    alias /home/www/kindle2flomo;
    index index.html;
    tryfiles $uri $uri/ /index.html;
}
location /md2wechat {
    alias /home/www/md2wechat;
    index index.html;
    tryfiles $uri $uri/ /index.html;
} 
...
```

首页是一个，kindle2flomo 是一个，md2wechat 是一个，后续可能还会有其他的静态页面需要代理。

### CDN 缓存

重新部署的时候，出现了一些小状况，耽误了一些时间。

nginx 的配置倒是配置完了，可是部署到线上的时候，总是不生效。起初一直以为是 nginx 的配置写得有问题，来回调整，浪费很多时间。

后来才意识到是 CDN 缓存的问题，试了一下，把 nginx 服务都给停了，网页还能正常访问，确认就是它的原因。

在腾讯云控制台一顿找，发现如下配置，刷新了下 URL，几分钟过后，就解决问题了。

![](/image/2022-12-04-week-summary/E2332E15-C46E-4DA9-8B20-3C6274B93404.43e2e6f26fbb4427ae5e468447542312.jpg)

CDN 是个好东西，访问速度提升非常明显，但是，对开发来说，也太不方便了。后续该完善一下独立开发、测试、部署的工作流程了。