---
title: "支付宝小程序导航栏自定义方案"
date: 2023-07-20T12:01:58+08:00
description: "支付宝小程序导航栏自定义"
tags: [mp-alipay, uni-app]
featured_image: "/tech-blog/images/notebook.jpg"
categories: mp-alipay
comment : false
---


## 先看支付宝小程序文档怎么说

### 导航栏背景色

```
// 若在 app.json 中的 window 中配置，则对全局页面生效
"window": {
  "titleBarColor": "#FFFFFF"
},

// 若在 page.json 中配置，则对当前页面生效
{
  "titleBarColor": "#FFFFFF"
}

// 也可以使用 my.setNavigationBar() 设置当前页面的导航栏的背景色
my.setNavigationBar({
    backgroundColor: "#FFFFFF"
})
```

### 导航栏前景色（状态栏文字、按钮、以及标题的颜色）
```
// frontColor必须和backgroundColor一起设置才能生效；
// 导航栏背景色会影响导航栏前景色。 若只设置导航栏背景色，则当背景色为 #ffffff时，前景色为黑色，否则前景色为白色。
my.setNavigationBar({
    frontColor: '#ffffff',
    backgroundColor: '#000000'
})
```

https://opendocs.alipay.com/mini/007l6m?pathHash=3b919f4b: 
> 字体颜色、返回按钮颜色属于导航栏前景色。前景色仅支持 #ffffff 和 #000000。
> 可以通过 my.setNavigationBar 入参 frontColor 直接修改导航栏前景色（基础库 2.7.24 开始支持）
> 也可通过 `my.setNavigationBar` 入参 `backgroundColor` 间接影响导航栏前景色。即当 `backgroundColor` 为白色时，前景色为黑色；当 `backgroundColor` 为其他颜色时，前景色为白色。


## 两种方案

### 方案1: title 用支付宝原生的

#### 背景色不透明
默认就好。

#### 背景色透明

uniapp 的 pages.json 中设置：
```
{
  "path": "color",
  "style": {
    "navigationBarTitleText": "色彩",
    "enablePullDownRefresh": false,
    "navigationStyle": "custom",
    "navigationBarBackgroundColor": "#ffffff",
    "mp-alipay": {
      "transparentTitle": "auto",
      "titlePenetrate": "YES",
      "defaultTitle": "色彩2"
    }
  }
},
```

注意
1. 设置了 `"transparentTitle": "always",` 或者` "transparentTitle": "auto",` ，要自己处理页面的 padding：
![image-20230720114248859](/tech-blog/images/20230720-mp_alipay_navigation-bar.assets/image-20230720114248859.png)
2. 前景色受背景色影响，非`#ffffff`背景色前景色会变成白色，所以如果背景色设置为类似白色的颜色，title 就不可辨认了。

### 方案2: title 设置为空，页面自己渲染 title

uniapp 的 pages.json 中设置`defaultTitle`为空：
```
{
  "path": "color",
  "style": {
    "navigationBarTitleText": "色彩",
    "enablePullDownRefresh": false,
    "navigationStyle": "custom",
    "navigationBarBackgroundColor": "#ffffff",
    "mp-alipay": {
      "transparentTitle": "always",
      "titlePenetrate": "YES",
      "defaultTitle": ""
    }
  }
},
```

然后自己写代码渲染 title。

## 官方文档

1. [导航栏使用须知 - 支付宝文档中心](https://opendocs.alipay.com/mini/07535r?pathHash=a54e002c)
2. [导航栏行为升级开发者适配指南 - 支付宝文档中心](https://opendocs.alipay.com/mini/077mhs?pathHash=82b5f57f)
3. [导航栏 FAQ - 支付宝文档中心](https://opendocs.alipay.com/mini/007l6m?pathHash=3b919f4b)