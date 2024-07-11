---
title: "uni-app 多端项目接入阿里云 ARMS"
date: 2021-11-05T12:01:58+08:00
description: "uni-app 多端项目如何加入阿里云 ARMS"
tags: [app, web, arms]
featured_image: "/tech-blog/images/notebook.jpg"
categories: uni-app
comment : false
---

## 官方文档

- [App监控概述](https://help.aliyun.com/document_detail/137303.html)
- [前端监控文档导读](https://help.aliyun.com/document_detail/170905.html)
- [什么是ARMS前端监控？](https://help.aliyun.com/document_detail/58652.html)

## vue项目如何引入

- [alife-logger - npm](https://www.npmjs.com/package/alife-logger)
- [如何引入阿里云ARMS前端监控？ · Issue #102 · FrankKai/FrankKai.github.io · GitHub](https://github.com/FrankKai/FrankKai.github.io/issues/102)
- [GitHub - rochzp/vue-arms: 基于阿里ARMS的Vue插件](https://github.com/rochzp/vue-arms)

## uniapp如何引入

1. H5：同vue项目处理方式。
2. 小程序：小程序方式。
3. app：和小程序处理方式一致。然后request那里是无法自动上报的，要手动上报。

- [uniapp开发的微信小程序可以集成阿里云arms前端监控吗](https://coding.m.imooc.com/questiondetail?cid=433&qid=223708)
- [Taro(mpvue/uniapp)引入Arms阿里前端监控 - Sushome](https://sushome.us/front-end/304/303/)
- [开始监控微信小程序](https://help.aliyun.com/document_detail/103992.html?spm=a2c4g.11186623.6.614.5ca47408b0MdQN)
- [webfunny前端监控平台新增功能教程：获取小程序、uni-app探针安装 - 掘金](https://juejin.cn/post/6996623986889064456)
- [(9条消息) 原生h5添加阿里云ARMS前端监控__小郑有点困了的博客-CSDN博客](https://blog.csdn.net/qq_44706619/article/details/115321552)
- [生命周期 - uni-app官网](https://uniapp.dcloud.io/collocation/frame/lifecycle?id=%e9%a1%b5%e9%9d%a2%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f)