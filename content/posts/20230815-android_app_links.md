---
title: "短信链接拉起 app 方案"
date: 2023-08-15T12:01:58+08:00
description: "iOS 上采用 Universal Links，Android 上采用 URL Scheme(非https/http)"
tags: [app]
featured_image: "/tech-blog/images/notebook.jpg"
categories: app
comment : false
---

## 需求

短信链接拉起app，去到app的某个特定页面。

## 结论

iOS 上采用 Universal Links，Android 上采用 URL Scheme(非https/http)。落地页统一使用 Universal Links 指向的 app 下载页。如果考虑到用户可能没有安装 app，那 https://www.kerryplus.com/ulink/ 需要返回页面（一般就是一个app下载页），这个页面上再执行 android 端的 url scheme 拉起app，类似这样的代码：

```javascript
  /**
   * 检查是否从短信中需拉起app的链接进入，如果是，且在 Anroid 上，则尝试拉起 app
  */
  function checkAppDeepLink() {
    const platform = getMobileOperatingSystem();
    const href = location.href;
    const index = href.indexOf('/ulink/');
    if (index > -1) {
      const param = href.substring(index + 7);// 7 就是 '/ulink/'.length

      // iOS 上面进入页面时 Universal Links 就已经打开 app了
      // 所以这里只用处理 Android
      if (platform == "Android") {
        location.href = "kerryplusenterprise://ulink/" + param;
      }
    }
  }
```

## iOS 测试结果

方案：Universal Links

| 默认浏览器 | 短信中用直链           | 短信中用短链接                            |
| ---------- | ---------------------- | ----------------------------------------- |
| Safari     | 直接拉起 app，参数正确 | 先跳转到 Safari，然后再拉起 app，参数正确 |
| Chrome     | 直接拉起 app，参数正确 | 先去到 Chrome，然后 404 了                |

直链测试短信：Hello，这是跳转到 ios app 的链接：https://www.kerryplus.com/ulink/?a=1，点击可以打开C端App。

短链测试短信：Hello，这是跳转到 ios app 的短链接：sh7x.cn/3000CNF，点击可以打开C端App。

方案 Url Scheme：

Safari 和 Chrome 都可以，拉起前会有询问（iOS上短信链接拉起小程序用的就是 universal links +  url scheme 拉起微信）。但是如果用户没有装 app，会有一个「打不开网页，因为网址无效」的弹窗。

- [iOS 深度跳转（scheme、universal link）-极光社区](https://community.jiguang.cn/article/464296)
- [深度跳转-scheme-极光社区](https://community.jiguang.cn/article/464337)
- [趣谈 iOS Universal Link - 知乎](https://zhuanlan.zhihu.com/p/444908355)


** Chrome+短链接 404 的原因是因为目标链接本身就是一个不存的地址，需要 nginx 做一下转发，返回app下载页 **

## Android 测试结果

方案：URL Scheme 和 App Links

**注意：URL Scheme 不能有大写字母！！**

**注意：intent-filter 要加到 android.intent.category.LAUNCHER 那个 activity 里面！！**

```xml
<!-- 方案1: URLScheme - https/http -->
            <!-- <intent-filter>
              <category android:name="android.intent.category.DEFAULT" />
              <category android:name="android.intent.category.BROWSABLE" />
              <action android:name="android.intent.action.VIEW" />
              <data android:scheme="http" android:host="app-links.dc1979.com" android:pathPattern=".*"/>
              <data android:scheme="https" android:host="app-links.dc1979.com" android:pathPattern=".*"/>
            </intent-filter> -->
            <!-- 方案2: URLScheme - 非https/http -->
            <intent-filter>
              <category android:name="android.intent.category.DEFAULT" />
              <category android:name="android.intent.category.BROWSABLE" />
              <action android:name="android.intent.action.VIEW" />
              <data android:scheme="h56131bcc" />
            </intent-filter>
            <!-- 方案3: App Links -->
            <!-- <intent-filter android:autoVerify="true">
              <category android:name="android.intent.category.DEFAULT" />
              <category android:name="android.intent.category.BROWSABLE" />
              <action android:name="android.intent.action.VIEW" />
              <data android:scheme="https" android:host="app-links.dc1979.com" android:pathPrefix="/ulink" />
            </intent-filter> -->
```

|                                                 | 测试1                                                        | 测试 2                                                       | 测试 3                                                       | 测试 4                                                     | 测试 5                                                     |
| ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- | ---------------------------------------------------------- |
| 方案                                            | URLScheme(https/http) + 直链                                 | URLScheme(https/http) + 短链                                 | URLScheme(非https/http) + 短链                               | App Links + 直链                                           | App Links + 短链                                           |
| 使用的 apk                                      | KerryPlus-3521-20230822.1434.apk                             | 同测试1                                                      | KerryPlus-DEV-3531-20230822.1439.apk                         | KerryPlus-DEV-3541-20230822.1442.apk                       | 同测试4                                                    |
| 短信内容                                        | Hello，这是跳转到 android app 的链接：https://app-links.dc1979.com/ulink/?a=1，点击可以打开C端App。 | Hello，这是跳转到 android app 的短链接：sh7x.cn/3000Fyl，点击可以打开C端App。 | Hello，这是跳转到 android app 的短链接：sh7x.cn/3000L48，点击可以打开C端App。 | 同测试1的短信内容                                          | 同测试2的短信内容                                          |
| OPPO RENO7, android 13, 手机默浏览器            | 浏览器打开链接，app未拉起                                    | 浏览器打开链接，app未拉起                                    | 浏览器打开链接，三秒后弹窗，用户点击打开后app成功拉起，参数正确✅ | 浏览器打开链接，app未拉起                                  | 浏览器打开链接，app未拉起                                  |
| VIVO S15 Android 13 手机默认浏览器              | 浏览器打开链接,未弹出弹窗，app未拉起                         | 浏览器打开链接,未弹出弹窗，app未拉起                         | 浏览器打开链接，需要手动点击 立即跳转，成功跳转至APP，参数正确☑️ | 浏览器打开链接，app未拉起                                  | 浏览器打开链接，app未拉起                                  |
| HUAWEI HarmonyOS 3.0.0 手机默认浏览器           | 浏览器打开链接,未弹出弹窗，app未拉起                         | 浏览器打开链接,未弹出弹窗，app未拉起                         | 浏览器打开链接，需要手动点击 立即跳转，弹出“打开”，点击“打开”，成功跳转至APP，参数正确☑️ | 浏览器打开链接，手动点击 立即跳转，未弹出弹窗，app未拉起   | 浏览器打开链接,未弹出弹窗，app未拉起                       |
| XIAOMI  11 MIUI 13.0.9 Android12 手机默认浏览器 | 浏览器打开链接,未弹出弹窗，app未拉起                         | 浏览器打开链接,未弹出弹窗，app未拉起                         | 浏览器打开链接，三秒后弹窗，用户点击打开后app成功拉起，参数正确✅ | 浏览器打开链接,未弹出弹窗，app未拉起                       | 浏览器打开链接,未弹出弹窗，app未拉起                       |
| ONEPLUS 6, android 10, 手机默认浏览器           | 短信界面弹窗，用户选择app打开后，app成功拉起，但是参数不对   | 浏览器打开链接，app未拉起                                    | 浏览器打开链接，三秒后弹窗，用户点击打开后app成功拉起，参数正确✅ | 短信界面弹窗，用户选择app打开后，app成功拉起，但是参数不对 | 浏览器打开链接，app未拉起                                  |
| ONEPLUS 6, android 10, Chrome                   | 短信界面弹窗，用户选择app打开后，app成功拉起，但是参数不对   | 短信界面弹窗，用户选择app打开后，app成功拉起，但是参数不对   | 浏览器打开链接，三秒后弹窗，用户点击打开后app成功拉起，参数正确✅ | 短信界面弹窗，用户选择app打开后，app成功拉起，但是参数不对 | 短信界面弹窗，用户选择app打开后，app成功拉起，但是参数不对 |



## uniapp 代码
```javascript
// onShow in App.vue 
try {
      const obj = { args: plus.runtime.arguments, '来源': plus.runtime.launcher };
      // ios: {"args":"https://www.kerryplus.com/ulink/testpage","来源":"uniLink"}
      // android:
      // eslint-disable-next-line no-console
      console.log('===', JSON.stringify(obj));
      setTimeout(() => {
        uni.showToast({
          title: JSON.stringify(obj),
          icon: 'none',
          duration: 1000 * 6
        });
      }, 500);
    } catch { }
```

## links

### 必读
- [【精选】可能是最全的h5唤起App方案_h5唤醒app-CSDN博客](https://lvan-zhang.blog.csdn.net/article/details/121210350)
- [suanmei/callapp-lib: 🔥call app from h5（H5唤起客户端 ）](https://github.com/suanmei/callapp-lib?tab=readme-ov-file)
- [H5唤起APP指南 | 拾壹小筑](https://suanmei.github.io/2018/08/23/h5_call_app/)

### app links
- [Android DEPPLINK、APPLink 原理简析-CSDN博客](https://blog.csdn.net/weixin_38754349/article/details/107724355)
- [uniApp实现h5页面唤醒app_h5 打开uniapp-CSDN博客](https://blog.csdn.net/wupinlong/article/details/109953480)
- [ios配置完universal links，如何通过universal links跳转到APP内指定页面？ - DCloud问答](https://ask.dcloud.net.cn/question/100042)
- [【报Bug】IOS下plus.runtime.arguments不能获取到URL Scheme协议参数【已解决】 - DCloud问答](https://ask.dcloud.net.cn/question/95040)
- [面试官：同学，说说 Applink 的使用以及原理 - DCloud问答](https://ask.dcloud.net.cn/article/36721)
- [处理 Android 应用链接  |  Android 开发者  |  Android Developers](https://developer.android.com/training/app-links?hl=zh-cn)
- [Debugging App Links in Android 12 🔗](https://zarah.dev/2022/02/08/android12-deeplinks.html)
- [语句列表生成器和测试器  |  Google Digital Asset Links  |  Google for Developers](https://developers.google.com/digital-asset-links/tools/generator?hl=zh-cn)
- [添加 Android App Links  |  Android Studio  |  Android Developers](https://developer.android.com/studio/write/app-link-indexing?hl=zh-cn)
- [android app links android app links 自动验证失效 1024_mob6454cc63f2dd的技术博客_51CTO博客](https://blog.51cto.com/u_16099183/7065726)
- [Web 页面打开 APP 方式总结 | Lleo](https://lleohao.github.io/2020/11/12/Web%20%E9%A1%B5%E9%9D%A2%E8%B0%83%E7%94%A8%20APP%20%E6%96%B9%E5%BC%8F%E6%80%BB%E7%BB%93/)
- [Android的Applink原理解析-8.0源码-CSDN博客](https://blog.csdn.net/fox_wei_hlz/article/details/102471312)
- [Oppo手机无法支持deeplink ，applink唤起应用的问题 - 适配支持 - 开发者社区](https://open.oppomobile.com/bbs/forum.php?mod=viewthread&tid=5205&extra=page%3D1&aid=46131)
- [一切为了运营！如何从推广短信链接唤起 App？ - 知乎](https://zhuanlan.zhihu.com/p/37303948)
- [Android 通过短信跳转到特定 App -- Android App Links - 简书](https://www.jianshu.com/p/213557eba7a3)

### url scheme
- [app-android-schemes - uni-app官网](https://uniapp.dcloud.net.cn/tutorial/app-android-schemes.html#)
- [UrlSchemes H5 安卓 无法打开 - Google 搜索](https://www.google.com/search?q=UrlSchemes+H5+%E5%AE%89%E5%8D%93+%E6%97%A0%E6%B3%95%E6%89%93%E5%BC%80&newwindow=1&sca_esv=556959823&biw=1888&bih=937&ei=L9LaZKrHIafy0PEPuZaHiAE&ved=0ahUKEwjqmriNv92AAxUnOTQIHTnLAREQ4dUDCA8&uact=5&oq=UrlSchemes+H5+%E5%AE%89%E5%8D%93+%E6%97%A0%E6%B3%95%E6%89%93%E5%BC%80&gs_lp=Egxnd3Mtd2l6LXNlcnAiIVVybFNjaGVtZXMgSDUg5a6J5Y2TIOaXoOazleaJk-W8gDIJECEYoAEYChgqMgcQIRigARgKMgcQIRigARgKSMpXUKMGWJJRcAl4AJABAJgBiQSgAY8zqgEJMi05LjguMS4yuAEDyAEA-AEBwgIKEAAYCBgeGA0YCsICCBAAGAgYHhgNwgIIEAAYiQUYogTCAgUQABiiBOIDBBgBIEGIBgE&sclient=gws-wiz-serp)
- [uni-app的H5如何跳转到App - DCloud问答](https://ask.dcloud.net.cn/question/65867)
- [H5页面唤起指定app或跳转到应用市场 - 简书](https://www.jianshu.com/p/21380058d609)
- [【报Bug】uni-app小米手机无法唤起app - DCloud问答](https://ask.dcloud.net.cn/question/83564)
- [android 配置的schemes不起作用唤醒不了 - DCloud问答](https://ask.dcloud.net.cn/question/175893)
- [URL schemes 按官方文档设置无效 - DCloud问答](https://ask.dcloud.net.cn/question/47639)
- [关于h5的安卓跳转app的UrlSchemes问题 - DCloud问答](https://ask.dcloud.net.cn/question/157732)
- [uniapp UrlSchemes配置 完成后使用浏览器无法打开 - DCloud问答](https://ask.dcloud.net.cn/question/118293)
- [Android配置Scheme使用浏览器唤起APP的方式，以及不生效问题解决-CSDN博客](https://blog.csdn.net/hwb04160011/article/details/106443382)
- [uni-app项目Android离线打包UrlSchemes设置_uniapp离线打包配置scheme-CSDN博客](https://blog.csdn.net/xiaohui_brook/article/details/113241717)

### test
- [.well-known gets ignored - Support - Netlify Support Forums](https://answers.netlify.com/t/well-known-gets-ignored/44897)
- [How to add a .well-known folder to my website - Support - Netlify Support Forums](https://answers.netlify.com/t/how-to-add-a-well-known-folder-to-my-website/28149)
- [app-links.dc1979.com/.well-known/assetlinks.json](https://app-links.dc1979.com/.well-known/assetlinks.json)
- [signal.group/.well-known/assetlinks.json](https://signal.group/.well-known/assetlinks.json)
- [Android App](https://ws-srv.glitch.me/app-links.html)


## iOS 上短信链接拉起微信小程序

测试短信：【嘉里建设】首页 https://wxaurl.cn/NAuYl9Mi2wo

文档：https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/sms.html

看起来混合利用了 Universal Links 和 Url Scheme。
