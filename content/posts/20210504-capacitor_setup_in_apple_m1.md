---
title: "Capacitor setup in Apple M1 Mac"
date: 2021-05-04T12:01:58+08:00
description: "如何在 M1 芯片的 MacBook 上配置 Capacitor 开发"
tags: [app, capacitor]
featured_image: "/tech-blog/images/notebook.jpg"
categories: app
comment : false
---

## 运行环境
macOS Big Sur 11.2.2
Xcode 12.4
Android Studio 4.1.3
node 14.16.1
Capacitor 2.4.1
Vue 3.0

## Web
```
npm install -g @ionic/cli
git clone https://github.com/ionic-team/tutorial-photo-gallery-vue.git
cd photo-gallery-capacitor-vue
npm install
ionic serve
```

## iOS

### 修改appId
git clone下来的代码里面的appId是`io.ionic.demo.pg.vue`，要换成自己的。VS Code打开代码文件夹，查找替换该appId为新的appId（比如`com.dc1979.capacitor.demo.vue`）

### 安装arm版CocoaPod
参考的是这个SO：https://stackoverflow.com/a/66556339
1. 确认安装了arm版Homebrew：`which brew`返回的结果是 `/opt/homebrew/bin/brew`。不是的话就按照这篇文章去安装arm版Homebrew：[在 M1 芯片 Mac 上使用 Homebrew](https://sspai.com/post/63935)
2. 安装arm版ruby：`brew install ruby`
3. 把环境变量加到PATH中：`echo 'export PATH="/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/3.0.0/bin:$PATH"' >> ~/.zshrc`
4. Execute `source ~/.zshrc` or restart your shell
5. 确定`which ruby`返回的是`$(brew --prefix)/opt/ruby/bin/ruby`
6. Install CocoaPods with `sudo gem install cocoapods`
7. Make sure you are using the correct pod binary by executing `which pod` (should be `$(brew --prefix)/lib/ruby/gems/3.0.0/bin/pod`)
8. Make sure ethon is version 0.13.0 or more with `gem info ethon`, otherwise run `sudo gem install ethon`

### 安装pod依赖
1. 运行`cd ios/App/`
2. 运行`pod install`

### 在Xcode中运行项目
1. 在项目根目录下运行`ionic cap sync`。可能最后会报错说sh pod install不成功，忽略它即可，因为我们上一步已经执行了pod install
2. `ionic cap open ios`
3. 在打开的Xcode中，登录自己的Apple ID，选择Automatically manage signing（如果有开发者账号的话，就选对应的profile）
4. 运行到模拟器或者真机

### live reload
1. 在项目根目录下运行`ionic cap run ios -l --external`
2. 到Xcode中运行项目到模拟器或者真机

### 使用Safari Web Inspector来调试iOS手机上的app
因为Capacitor app本质上就是一个WebView，所以可以用Safari Web Inspector来调试Capacitor app里面的代码。

1. 在iPhone或者iPad上 启用「网页检查器」：进入 设置 > Safari 浏览器> 高级 中，启用「网页检查器」
2. 在mac电脑上，打开Safari，在 Safari > Preferences > Advanced 中，勾选 Show Develop menu in menu bar。重启Safari。
3. 将iPhone或者iPad连接到mac电脑上，运行Capacitor app，在Safari的 Develop > [你的iOS设备名称]下面，点击需要调试的页面（一般就是localhost - xxx），打开Web Inspector。

## Android

### 安装android sdk
安装Android Studio，打开它，安装android sdk（需要先配置代理翻墙）

### 在Android Studio中运行项目
1. 因为在iOS部分我们已经运行过`ionic cap sync`，所以android project其实已经准备好了
2. `ionic cap open android`
3. 在打开的Android Studio中，运行项目到真机（模拟器暂时在M1上还不行，只有preview版本）

### live reload (iOS live reload没运行)
1. 在项目根目录下运行`ionic cap run android -l --external`
2. 到Android Studio中运行项目到真机

### live reload (iOS live reload已运行)
1. 新开一个terminal窗口
2. 在项目根目录下运行`npx cap copy android&&npx cap open android`
2. 到Android Studio中运行项目到真机

### 使用Chrome DevTools来调试Android手机上的app
因为Capacitor app本质上就是一个WebView，所以可以用Chrome DevTools来调试Capacitor app里面的代码。

1. 在Android手机上的开发者选项中启用USB调试
2. 打开Chrome，在地址栏中输入`chrome://inspect#devices`，回车
3. 在Chrome Devices页面上查看Discover USB devices是否已经勾选，没有的话勾选上
4. 将Anroid手机连接到电脑，Android手机上会询问「是否允许USB调试」，选择「允许」
5. 此时该Android手机应该会出现在Chrome Devices页面上，点击对应页面下面的「inspect」链接，就可以看见和Web开发时一样的DevTools页面了

更多信息可以参考Chrome官方的文档：[Remote debug Android devices - Chrome Developers](https://developer.chrome.com/docs/devtools/remote-debugging/)

### 使用IntelliJ IDEA CE ? 
因为Android Studio暂时还不支持Apple M1（20210504），网上有人建议用IntelliJ IDEA CE来编辑和运行android项目，暂时还没测试。
安装IntelliJ IDEA CE后，第一件事情也是配置代理翻墙。

## 真实的Capacitor项目在Apple M1上可以正常build吗
测试了Clubhouse App，可以。

## links
- [M1芯片Mac搭建前端开发环境 - 简书](https://www.jianshu.com/p/ef16b3bec42b)
- [Apple Silicon 移动端开发填坑记录（持续更新） - 知乎](https://zhuanlan.zhihu.com/p/345284558)
- [node.js - Node Version Manager install - nvm command not found - Stack Overflow](https://stackoverflow.com/questions/16904658/node-version-manager-install-nvm-command-not-found)
- [Tutorial: Create your first Android application | IntelliJ IDEA](https://www.jetbrains.com/help/idea/create-your-first-android-application.html#Android-make-application-interactive)
- [Speed up Android Compilation in Apple M1 Device | by Elye | Mobile App Development Publication | Apr, 2021 | Medium](https://medium.com/mobile-app-development-publication/speed-up-android-compilation-in-apple-m1-device-6dbca65b4bb9)

