---
title: "微信小程序持续集成相关"
date: 2021-07-15T12:01:58+08:00
description: "预览和上传的区别；一个奇怪的bug；微信小程序强制更新"
tags: [mp-weixin]
featured_image: "/tech-blog/images/notebook.jpg"
categories: mp-weixin
comment : false
---

## 预览和上传的区别

微信开发者工具有两个按钮：预览和上传。

[miniprogram-ci](https://www.npmjs.com/package/miniprogram-ci) 是从[微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/devtools.html)中抽离的关于小程序/小游戏项目代码的编译模块，有 `ci.preview`和`ci.upload`两个方法。和上两个按钮对应。

预览时，只能在小程序助手的开发版列表中看到，在PC Web小程序后台的开发版列表看不到。
上传时，只能在PC Web小程序后台的开发版列表看到，在小程序助手的开发版列表中看不到。

只有在PC Web小程序后台才可进行「选为体验版本」及「提交审核」操作。

所以现在的做法是在preview.js中先执行一次`ci.preview`，再执行一次`ci.upload`。

### links

- [用__wxConfig.envVersion区分小程序体验版，开发板，正式版 | 微信开放社区](https://developers.weixin.qq.com/community/develop/article/doc/000e6606ab4ac0edb4791eb4951013)
- [小程序助手里面的版本查看的开发版不能自动更新啊 | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/00062c56858b5886efd76ffc55ac00)
- [使用 miniprogram-ci 上传的版本在小程序助手里看不见？ | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/00040864e180707313ba0503951400)
- [微信开发者工具交流专区 | 微信开放社区](https://developers.weixin.qq.com/community/minihome/mixflow/1213999125012856832)
- [可否在小程序助手里提供发布版本的功能? | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/0002ae57f98a00288cc9303355b400?_at=1626747445453)
- [开发者工具上传后，小程序助手没有该版本显示？ | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/0000662ff9ca982a1f9a166885c000?_at=1626747511355)


## 一次奇怪的bug

有一天发现线上的小程序有奇怪的样式问题，同样的代码在我们自己电脑上打包传上去却没有问题。最后发现是jenkins上的pipeline里面使用的miniprogram-ci使用的版本太旧了。se照着package.json来确定的版本号，但是在我们本地，我们用的yarn.lock中的版本。


## 微信小程序强制更新

从文档可以看出，判断小程序是否有新版本和下载小程序新版本代码都是由微信自行负责，我们的代码无法介入。我们的代码能够做的就是：「如果需要马上应用最新版本，可以使用 [wx.getUpdateManager](https://developers.weixin.qq.com/miniprogram/dev/api/base/update/wx.getUpdateManager.html) API 进行处理。」

`updateManager.applyUpdate()`会有一次系统提示框，所以我们自己的代码就不需要提示用户了。

### links

- [微信小程序发布新版本时自动提示用户更新 - 逍遥云天 - 博客园](https://www.cnblogs.com/xyyt/p/10616730.html)
- [uni-app 小程序更新后强制用户升级 - 瞭月](https://www.lervor.com/archives/159/)
- [(3)强制更新 | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/000c2430d30b70251e86f0a0256c09)
- [小程序更新机制 | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/framework/runtime/update-mechanism.html)
- [UpdateManager | 微信开放文档](https://developers.weixin.qq.com/minigame/dev/api/base/update/UpdateManager.html)
- [https://developers.weixin.qq.com/community/search?query=UpdateManager&block=1&blogcategory=0&sort=0&time=0&starttime=&endtime=&type%5Bpage%5D=1&type%5Bblock%5D=1&type%5Btype%5D=1&type%5Bquery%5D=UpdateManager&page=2](https://developers.weixin.qq.com/community/search?query=UpdateManager&block=1&blogcategory=0&sort=0&time=0&starttime=&endtime=&type%5Bpage%5D=1&type%5Bblock%5D=1&type%5Btype%5D=1&type%5Bquery%5D=UpdateManager&page=2)
- [为什么小程序更新提示只有ios用户有，安卓用户看不到更新提示？ | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/00040e393585701cd9ccc390b56c00?highLine=UpdateManager)
- [这几天小程序的updateManager.onCheckForUpdate 安卓不能用了，苹果好使 | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/000ce49fbfcba890f4bc98baa5a400?highLine=UpdateManager)
- [版本更新提醒，工具上好用，真机就没反应？ | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/0004044445c7d09054bcd55ee5b800?highLine=UpdateManager)
- [updateManager.applyUpdate() 调用强制更新后，本地storage丢失 | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/0004a03ae2ca80a66bfcd16e156800?highLine=UpdateManager)
- [小程序版本更新,本地缓存会被清理 | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/0002eaddc8caf0e5a21ce74bd51800)
- [开发过程中怎么样可以出发 UpdateManager.applyUpdate() 更新逻辑? | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/000e86ed5f8318d7ce0c1b5f75bc00)
- [UpdateManager 更新，没有提示弹出弹出框，直接就是新版本呀？ | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/000ec2bbe20430ca076c55d5f5b000)
- [wx.getUpdateManager是检测代码改动还是检测版本号？ | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/000020a4584b3831588a08fb95b800?_at=1637469181959)
- [wx.getUpdateManager是检测代码改动还是检测版本号？ | 微信开放社区](https://developers.weixin.qq.com/community/develop/doc/000684d73407b0e994a96050856400)

