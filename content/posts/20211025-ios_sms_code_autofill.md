---
title: "iOS上面的验证码自动填充功能"
date: 2021-10-25T12:01:58+08:00
description: "验证码自动填充；自定义的验证码输入组件"
tags: [web, ios]
featured_image: "/tech-blog/images/notebook.jpg"
categories: web
comment : false
---

测试页面在这里：[Demo](https://stackblitz.com/edit/b5a3c-html5-css3?file=input.html)

## 结论

1. 只有iOS上才有验证码自动填充功能吗？对。
2. 在iOS上，只有纯数字验证码才能自动填充，数字字母验证码没法自动填充？对。
3. 在iOS上，非Safari浏览器上，自动填充会填写两次？[H5 页面在 IOS 上短信验证码点击一次会自动填充两次 bug 解决 - 海滨擎蟹](https://www.seasidecrab.com/Web/742.html)
4. 修改自动填充后input的样式：[Change Autocomplete Styles in WebKit Browsers | CSS-Tricks](https://css-tricks.com/snippets/css/change-autocomplete-styles-webkit-browsers/) 注意字体样式要根据各个输入框去调整
5. 验证码输入控件处理：不会触发paste事件，触发的事keyup和input事件

android手机上的自动填充？和QA的对话：
> 对于android系统，目前不会有这个东西。大部分android手机，都是会在屏幕顶部的区域出现短信的提示，短信提示中会有一个复制按钮，点这个复制按钮验证码被复制到剪贴板，然后到输入框中粘贴。我找用android机的同事问了一圈，目前荣耀、一加、小米都是这个行为。现在这个四个格子的验证码输入控件也是支持粘贴的，你可以再测试一下。

## 验证码输入组件在android机上有问题

最早是用一加6发现keycode一直为0，后面做了调整：

- [javascript - Keycode is always zero in Chrome for Android - Stack Overflow](https://stackoverflow.com/questions/17139039/keycode-is-always-zero-in-chrome-for-android)
- [KeyboardEvent.code - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code)
- [event.which | jQuery API Documentation](https://api.jquery.com/event.which/)

后来QA报bug，说在华为和荣耀上验证码输入控价都无法使用，因为keyCode完全不对：

- [javascript - keyCode on android is always 229 - Stack Overflow](https://stackoverflow.com/questions/36753548/keycode-on-android-is-always-229)
- [(9条消息) 中文输入法时获取e.keycode一直为229_brilliantSt的博客-CSDN博客](https://blog.csdn.net/brilliantSt/article/details/115381443)
- [(9条消息) 解决安卓端拿不到软键盘真实keycode值，全为229。_所有热爱的东西都要不余遗力-CSDN博客](https://blog.csdn.net/qq_45327292/article/details/103274456?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7Edefault-5.essearch_pc_relevant&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7Edefault-5.essearch_pc_relevant)
- [(9条消息) 移动端H5页面关于软键盘的一些踩坑记录_this_ITBoy的博客-CSDN博客](https://blog.csdn.net/this_ITBoy/article/details/100089299?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.essearch_pc_relevant&spm=1001.2101.3001.4242.1)

最后没办法，只能改为在android机器上就不显示验证码输入控件，用一个普通文本框来替代。

## links

- [用起来像 UITextField 一样的自定义验证码输入框 - 掘金](https://juejin.cn/post/7001860958461100040)
- [记一次验证码输入框的实现 | kemchenj](https://kemchenj.github.io/2019-04-07/)
- [(3条消息) h5短信验证码复制/自动填充等问题_阿之阿佐-CSDN博客](https://blog.csdn.net/u010214074/article/details/116313002)
- [One Time Security Code in iOS Safari | by Mark de Vries | Medium](https://medium.com/@lamalamaMark/one-time-security-code-in-ios-safari-27869e16085f)
- [ios h5 验证码填充 - 掘金](https://juejin.cn/post/6844903919404089357)
- [(5条消息) 项目总结之ios手机中input框中粘贴短信验证码keyup事件无效_时清云的博客-CSDN博客](https://blog.csdn.net/xiaolinlife/article/details/90477430)

