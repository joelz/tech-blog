---
title: "vue web app 接入 Azure AD SSO"
date: 2021-12-08T12:01:58+08:00
description: "vue web app 接入 Azure AD SSO"
tags: [web, sso, azure]
featured_image: "/tech-blog/images/notebook.jpg"
categories: web
comment : false
---


## 第三方应用接入Azure AD SSO流程
1. 确定账号体系的对应关系（SP 中的账号和 AAD 中的哪个字段对应）
2. 确定接入协议（SAML 还是 OAuth）
2. 确定 SP 中有哪些类型的应用要集成（SAP，Web App，Web API，Desktop，Mobile）
3. 确定接入方案
4. 新建 dev AAD 的application，创建配置
5. SP 方开发和测试
6. （可选）有些事项需要找香港同事确认
7. PROD AAD 配置
8. SP 方测试 PROD AAD 配置无误
9. 上线

## OAuth in Azure AD

SSO_in_Azure_AD_using_OAuth_OIDC_20220520.pptx

### default access token 是 jwt，但是无法验证签名  Invalid signature 
[validation - Invalid signature while validating Azure ad access token, but id token works - Stack Overflow](https://stackoverflow.com/questions/45317152/invalid-signature-while-validating-azure-ad-access-token-but-id-token-works)

### OIDC in OAuth
[Diagrams of All The OpenID Connect Flows | by Takahiko Kawasaki | Medium](https://darutk.medium.com/diagrams-of-all-the-openid-connect-flows-6968e3990660)

## SAML in Azure AD
- [What is SAML and how does SAML Authentication Work](https://auth0.com/blog/how-saml-authentication-works/)

### SAML 配置
- [How the Microsoft identity platform uses the SAML protocol - Microsoft identity platform | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-saml-protocol-reference)
-[SAML authentication with Azure Active Directory | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/auth-saml)
- [通过SAML集成Azure AD](https://help.aliyun.com/document_detail/312185.html)
- [访问管理 Azure Active Directory 单点登录腾讯云指南-用户指南-文档中心-腾讯云-腾讯云](https://cloud.tencent.com/document/product/598/35713)

### SAML in MSAL.js?
- [What about supporting SAML 2.0 in this library? · Issue #329 · AzureAD/microsoft-authentication-library-for-java](https://github.com/AzureAD/microsoft-authentication-library-for-java/issues/329)
- [jwt - Acquire a SAML token in MSAL.js - Stack Overflow](https://stackoverflow.com/questions/63495395/acquire-a-saml-token-in-msal-js)：
> Also ,[Single Sign-On SAML protocol](https://docs.microsoft.com/en-us/azure/active-directory/develop/single-sign-on-saml-protocol) and [Federated Authentication with a SAML Identity Provider](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-fed-saml-idp) should be good starting points to implement SAML directly.

### 例子
例子（里面还有用azure saml 的示例，部署在 https://aad-saml.glitch.me/ ）：https://github.com/joelz/certificates-sample

asp.net 可以用的库：[jitbit/AspNetSaml: Very simple SAML 2.0 consumer module for ASP.NET/C#](https://github.com/jitbit/AspNetSaml?tab=readme-ov-file)

####  SAML 101
* [Azure Single Sign On SAML Protocol - Microsoft identity platform | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/single-sign-on-saml-protocol)
* [Quickstart: Set up SAML-based single sign-on (SSO) for an application in your Azure Active Directory (Azure AD) tenant | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-setup-sso)
* [Custom SAML in Azure AD | Postman Learning Center](https://learning.postman.com/docs/administration/sso/saml-in-azure-ad/)
* [Authenticate an Azure AD user with SAML for ASP.NET Core - Matthijs's Blog](https://matthijs.hoekstraonline.net/2020/04/14/authenticate-an-azure-ad-user-with-saml-for-asp-net-core/)
* [ASP.NET Core SAML Authentication with Azure AD](https://cmatskas.com/asp-net-core-saml-authentication-with-azure-ad/)
* [Azure Single Sign-On using SAML 2.0 Protocol and ASP.NET C# - Stack Overflow](https://stackoverflow.com/questions/55408087/azure-single-sign-on-using-saml-2-0-protocol-and-asp-net-c-sharp)


## 保护API
自定义scope，或者自定义 app roles。

[oauth 2.0 - How to control/set permissions for a single user when using Azure AD B2C - Stack Overflow](https://stackoverflow.com/questions/50128504/how-to-control-set-permissions-for-a-single-user-when-using-azure-ad-b2c)
[Add app roles and get them from a token - Microsoft identity platform | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)

👇下面的说法不对。

=====

拿到access_token 去访问 resource owner，怎么知道权限呢？scope。
kip 里面拿到acess_token，仍然是去找 microsoft graph 问他是谁，难道 ms graph 是一个 resource owner？对的。所以Azure Portal 中配置 Application 时需要在 API Permissions 中更加上MS Graph 相关的 Permission

自己的App（即resource owner，简称RO）中要加权限控制，整个配置的流程是这样的：
1. 为client app注册application：[Quickstart: Register an app in the Microsoft identity platform - Microsoft identity platform | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
2. 为RO配置权限。文档中叫Expose an API，可以理解为RO是一个大的API，不同权限对应不同的scope。添加scope：[Quickstart: Register and expose a web API - Microsoft identity platform | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-expose-web-apis)
3. 为RO添加的permission，包括了微软提供的一些permission（比如 MS Graph 的permission）和我们自己定义的scope对应的permission：[Quickstart: Configure an app to access a web API - Microsoft identity platform | Microsoft Doc](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-access-web-apis)


# S端集成Azure AD SSO

## 基础概念

1. Azure AD是微软提供的云服务，它是一个多租户系统。公司是Azure AD的一个租户（或者叫域），对应的租户名称是kerryprops.com。我们开发时会使用一个测试用的租户，租户名称为kpltest.onmicrosoft.com。
2. SSO：单点登录。整个单点登录过程中，会涉及到用户、身份提供商（Identity Provider，简称IdP）和服务提供商（Service Provider，简称SP）。在S端用户登录这个场景中，Azure AD 是 IdP，S端是SP同时也是IdP（因为我们有自己的Profile Service）。整个SSO的目的就是用Azure AD的token去换回S端的token（因为调用后端Service API使用的是S端的token，而非Azure AD的token）
3. 涉及到多个 IdP 的 SSO 有很多方式可以实现，比如 SAML 2.0，WS-Federation，OpenID Connect（OIDC）和 OAuth。S端是一个前后端分离的单页应用（SPA），采用的是[OAuth 2.0 Authorization code flow (with PKCE)](https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-spa-overview#overview)。页面请求顺序如下图：
![admin-portal-sso.png](./assets/admin-portal-sso.png)
4. MSAL库：Microsoft Authentication Library，微软官方出品的类库，可方便地处理使用Azure AD做SSO的各类场景。有各种语言的版本。JavaScript语言的版本叫 [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)


## Azure AD上的配置

这部分需要Azure AD的管理员才能配置。管理员登录到Azure AD Portal: https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade

> 如果想自己尝试配置和管理一个Azure AD的租户，可以用个人的微软账号（比如live.com/hotmail.com/outlook.com 邮箱账号）去申请[Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program)，可以得到一个免费的Azure AD租户。

## S端代码说明
使用的npm pacakge：[@azure/msal-browser - npm](https://www.npmjs.com/package/@azure/msal-browser)（就是上文说到的 MSAL.js ）

### 配置项
配置项写在 `src/store/modules/user.js` 中：
```javascript
    msalLoginRequest: {
      scopes: ['openid', 'profile', 'User.Read'], // 对应上文中的 API Permissions
    },
    msalUseRedirect: process.env.VUE_APP_AZURE_USE_REDIRECT == 'true', // 参见下文说明1
    msalConfig: { // 参见下文说明2
      auth: {
        clientId: getAzureConfig().clientId,
        authority: getAzureConfig().authority, // 里面有tenantId
        navigateToLoginRequestUrl: false,
      },
      cache: {
        cacheLocation: 'localStorage',
      },
    },
```

说明：
1. 这个选项是用来控制浏览器从S端登录页到Azure AD登录页时使用跳转还是弹窗。参考这里：https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-spa-sign-in?tabs=javascript2#choosing-between-a-pop-up-or-redirect-experience
2. Configuration Options完整文档在这里：https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/configuration.md

### 初始化 MSAL 实例
在 `src/main.js` 中：
```javascript
Vue.prototype.$msalInstance = new PublicClientApplication(
  store.state.user.msalConfig
);
```

### 登录
在 `src/views/login.vue` 中：
```javascript

  mounted() {
    // ....
    // 如果从S端登录页到Azure AD登录页时使用跳转方式，当用户在Azure AD登录成功时
    // 浏览器会跳转会 /login，同时url中会带上#code=xxxxx，
    // 从code的值中可用解析出Azure AD的token
    if (this.$store.state.user.msalUseRedirect) {
      const code = document.location.href.split('#').pop();
      if (code && code.indexOf('code=') === 0) {
        console.log(
          'AAD_SSO',
          'Find auth code in path, starting handleRedirectPromise...'
        );
        this.loading = true;
        this.$msalInstance
          .handleRedirectPromise()
          .then((resp) => {
            console.log(
              'AAD_SSO',
              'handleRedirectPromise on mounted done',
              resp
            );
            let accessToken = resp.accessToken;
            this.msalLogin(accessToken);
          })
          .catch((err) => {
            console.error(err);
            this.loading = false;
          });
      }
    }
  },
  methods: {
    // ...
    // acquireTokenSilent，成功则调用下面的msalLogin；
    // 失败则调用 getMsalAccessTokenViaInteractiveMode
    // 目前未用到
    async getMsalAccessToken(account) {
      this.loading = true;
      const request = { // // 参见下文说明1
        scopes: ['User.Read'],
        prompt: 'select_account',
      };

      try {
        let tokenResponse = await this.$msalInstance.acquireTokenSilent(
          request
        );
        console.log('setAccessToken', tokenResponse.accessToken);
        this.msalLogin(tokenResponse.accessToken);
      } catch (error) {
        console.error(error);
        console.error(
          'Silent token acquisition failed. Using interactive mode'
        );
        this.getMsalAccessTokenViaInteractiveMode();
      }
    },
    // 交互模式登录：弹窗或者跳转到Azure AD登录页面，完成登录，获取到Azure AD的token
    async getMsalAccessTokenViaInteractiveMode() {
      this.loading = true;
      // https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/request-response-object.md
      const request = {
        scopes: ['User.Read'],
        prompt: 'select_account',
      };

      console.info(
        'Using interactive mode to msal get access token...'
      );
      if (this.$store.state.user.msalUseRedirect) {
        this.$msalInstance.acquireTokenRedirect(request);
      } else {
        let tokenResponse = await this.$msalInstance.acquireTokenPopup(
          request
        );
        console.log(
          `Access token acquired via interactive auth ${tokenResponse.accessToken}`
        );
        this.msalLogin(tokenResponse.accessToken);
      }
    },
    // 调用profile serivce接口，用Azure AD的token去获取S端可用的token
    // 完成登录过程
    // Azure AD的token可以用微软提供的接口 https://graph.microsoft.com/v1.0/me 来验证
    async msalLogin(msalAccessToken) {
      this.$store
        .dispatch('MsalLogin', msalAccessToken)
        .then(() => {
          console.log('Msal login finally done!!!');
          // 。。。。
        })
        .catch((e) => {
          console.error(e);
          this.loading = false;
        });
    },
    // ...
  },   

```

说明：
1. Request 和 Response 对象的完整文档在这里：https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/request-response-object.md

### 登出
登出时不涉及Azure AD的登出，只是删除S端自己的token即可。

## 其他

### 登录报错：pkce not created
by yuyang in SKC。错误信息如下：
```
uncaught (in promise) BrowserAuthError:app.24961a1a.j5:141

pkce not created: The PKCE code challenge and verifier could not be generated. Detail TypeError: Cannot read property 'digest' of undefined at t les constructor@ (https: //static.kerryprops.com.cn/kip/kip-admin-portel/static/is/app.24361ale-j9:141:4139)

at newt (nttpst//statickerryprops.com.en/kip/kip-admin-portal/static/js/app.496181a.45/141/225355

at Function.t, createPkceNotGeneratedError (https://static.kerryprops.com.cn/kip/kip-admin-pontal/static/is/app.24961ala.js:141:22692)

at e-sanonymous>/(https://static.kerryprops.com.cn/kip/kip-admin-portal/
```
原因：
用户没有使用https链接登录S端。使用 https://admin.kerryplus.com 访问登录成功。

### 错误2
[How to fix AADSTS9002325: Proof Key for Code Exchange is required for cross-origin authorization code redemption error in msal react application. - Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/1194524/how-to-fix-aadsts9002325-proof-key-for-code-exchan)


## 参考链接

- [Vue.js User Authentication and the new Azure SDKs - Azure SDK Blog](https://devblogs.microsoft.com/azure-sdk/vue-js-user-authentication/)
- [Initialize MSAL.js client apps - Microsoft identity platform | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-js-initializing-client-applications)
- [Single-page app sign-in & sign-out - Microsoft identity platform | Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-spa-sign-in?tabs=javascript2)