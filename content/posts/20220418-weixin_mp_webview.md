---
title: "小程序及App内嵌H5"
date: 2022-04-18T12:01:58+08:00
description: "微信小程序、支付宝小程序和 App 中 webview 内嵌 H5 的一些问题。"
tags: [mp-weixin, mp-alipay, app]
featured_image: "/tech-blog/images/notebook.jpg"
categories: web
comment : false
---


## 微信小程序内嵌 H5
### 通信机制
#### 微信小程序向 H5 发送消息
1. URL search 传参：在 H5 初始加载时，通常会带上用户的登录态作为参数，使 H5 可以继续与后台交互。类似 `https://example.com/h5.html?token=SGVsbG9Xb3JkIQ==`
2. URL hash 传参：在 H5 浏览期间，可以 利用网页 URL 中的 hash 部分改变不会导致网页重新加载的特点，实现小程序向 H5 发送消息。但是：[【报Bug】UNIAPP H5修改hash模式的URL参数会导致页面重载，以前旧的hbx2.8.11是不会重载的 - DCloud问答](https://ask.dcloud.net.cn/question/125195)
3. websocket

#### H5 向微信小程序发送消息
1. postMessage：小程序内嵌 H5 可以使用 JSSDK 向小程序发送消息。**网页向小程序 postMessage 时，只会在特定时机（小程序后退、组件销毁、分享）触发并收到消息，不是实时触发。e.detail = { data }，data是多次 postMessage 的参数组成的数组**
2. 路由传参：在使用 JSSDK 调起小程序页面时，可以通过在页面路径后携带参数的方式通信。类似 `wx.miniProgram.navigateTo({ url: '/path/to/page?foo=bar' }) `
3. websocket

### 不足
1. 对于需要用户交互才能使用的小程序能力（比如分享，获取用户手机号，获取用户信息，获取用户头像，订阅消息等），需要跳转页面
2. WebView 页面的导航栏只能固定用默认的白底黑字风格

### 参考链接
- [微信小程序混合开发笔记](https://ymcai.xyz/posts/202107021145.html)
- [微信小程序+webview+vue混合开发 传值踩坑_choutian5268的博客-CSDN博客](https://blog.csdn.net/choutian5268/article/details/100810817)
- [【报Bug】UNIAPP H5修改hash模式的URL参数会导致页面重载，以前旧的hbx2.8.11是不会重载的 - DCloud问答](https://ask.dcloud.net.cn/question/125195)
[- web-view | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html)

## App内嵌 H5
### 通信机制
#### App向 H5 发送消息
1. [evalJS](https://www.html5plus.org/doc/zh_cn/webview.html#plus.webview.WebviewObject.evalJS)：
```typescript
const js =
  `window.settlePromise('resolve',` +
  `'${res.detail.data[0].data.paymentSessionId}',` +
  `'${JSON.stringify(result)}')`;
  (this.webview as any).evalJS(js);
```

#### H5 向App发送消息
1. uni.webView.postMessage：网页向应用发送消息，在 web-view 的 message 事件回调 event.detail.data 中接收消息。实时触发。
``` typescript
(uni as any).webView.postMessage({
      data: {
        action: 'PAY',
        data: {
          payOption: 'WECHATPAY',
          paymentSessionId: sessionId
        }
      }
    });
```

#### web-view 加载的 HTML 中可以调用html  5+ 的能力
比如HTML5+ 中的 [扫码](https://www.html5plus.org/doc/zh_cn/barcode.html)

### 不足
1. iOS App 侧滑会关闭整个webview

- [uniapp如何监听或者知道ios左滑返回事件？ - DCloud问答](https://ask.dcloud.net.cn/question/101333)
- [uni-app自定义返回逻辑教程 - DCloud问答](https://ask.dcloud.net.cn/article/35120)

### 参考链接

- [web-view | uni-app官网](https://uniapp.dcloud.io/component/web-view.html)
- [evalJS - HTML5+ API Reference](https://www.html5plus.org/doc/zh_cn/webview.html#plus.webview.WebviewObject.evalJS)

## 支付宝小程序内嵌 H5

### 通信机制

很方便，支持双向即时通信。

#### H5 端代码

首先需要在 `index.html` 中引入 js sdk：
```html
  <script>
    if (navigator.userAgent.indexOf('AliApp') > -1) {
      document.writeln('<script src="https://appx/web-view.min.js"' + '>' + '<' + '/' + 'script>');
    }
  </script>
  <script>
    // 网页向小程序 postMessage 消息
    // my.postMessage({ name: "测试web-view" });
    // 接收来自小程序的消息。
    my.onMessage = function (e) {
      console.log('message from webview: ',e); // {'sendToWebView': '1'}
    }
  </script>
```

在 vue 页面中发送消息：

```vue
<template>
  <view>
    <button @click="send">Send</button>
  </view>
</template>

<script>
	export default {
		methods: {
			send: function () {
                my.postMessage({name:"测试web-view", time: (new Date()).toString()});
            },
		}
	}
</script>
```

#### 支付宝小程序代码（uniapp）

```vue
<template>
  <view>
    <web-view id="web-view-1" :src="url" @message="onmessage"></web-view>
  </view>
</template>

<script>
	var app = getApp()
	export default {
		data() {
			return {
				url:''
			}
		},
		onLoad: function(options) {
			this.url = decodeURIComponent(options.url)
			setTimeout(() => {
				// 往 webview h5 发送一条消息
				const webViewContext = my.createWebViewContext('web-view-1');
				webViewContext.postMessage({
				  'event': 'onLoad',
				   'time': (new Date()).toString()
				});
			}, 1000*10);
		},
		onShareAppMessage(options) {
		    my.alert({ content: JSON.stringify(options.webViewUrl) });
		    return {
		      title: '分享 web-View 组件',
		      desc: 'View 组件很通用',
		      path: 'page/component/component-pages/webview/baidu',
		      'web-view': options.webViewUrl,
		    };
		  },
		methods: {
			// 接收从 webview h5 发来的消息
			onmessage(e) {
				console.log('===> message from webview h5:',e)
				setTimeout(()=>{
					// 5 秒后再往 webview h5 发送一条消息
					const webViewContext = my.createWebViewContext('web-view-1');
					webViewContext.postMessage({
					  'sendToWebView': '1',
					   'time': (new Date()).toString()
					});
				},1000*5);
			},
		}
	}
</script>
```

### 参考链接

- [web-view H5 页面承载 - 支付宝文档中心](https://opendocs.alipay.com/mini/component/web-view)
- [web-view 常见问题 - 支付宝文档中心](https://opendocs.alipay.com/mini/component/mg7rvg)
- [web-view | uni-app官网](https://uniapp.dcloud.net.cn/component/web-view.html#)

