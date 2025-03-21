---
layout: post
title: 我所理解的 Azure AD / Easy Auth 工作逻辑
subtitle:
categories: FrontEnd Web Azure OAuth
tags: [FrontEnd, Web, Azure, OAuth]
---

## 我所理解的 Azure AD / Easy Auth 工作逻辑

### Microsoft 官方文档

<https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization>

<https://learn.microsoft.com/en-us/azure/app-service/tutorial-auth-aad?pivots=platform-linux>

<https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-oauth-tokens>

### 文档中提到的架构图

![front-end-app-service-to-back-end-app-service-authentication]({{ "/assets/images/2025-03-20/front-end-app-service-to-back-end-app-service-authentication.png" | absolute url }})

![Architechture]({{ "/assets/images/2025-03-20/architecture.png" | absolute url }})

### 按我理解后扩充的架构图(本地调试模式)

![local]({{ "/assets/images/2025-03-20/local.jpg" | absolute url }})

### 按我理解后扩充的架构图(Azure 部署模式)

![brief]({{ "/assets/images/2025-03-20/brief.jpg" | absolute url }})

### 流程图

![sequence]({{ "/assets/images/2025-03-20/sequence.jpg" | absolute url }})

说明: 前后端服务对应的 Azure App Service 都启用了 Easy Auth

1. 用户访问前端服务 URL（例如 https://your-front-end-web-app.azurewebsites.net）。

2. Easy Auth Middleware（内置在 Azure App Service）拦截请求，检测到 没有 Session Token（Cookie 不存在或已过期）。

   - 返回 302 重定向到身份提供商（IdP）（Microsoft、Google、Facebook、X 等）。
   - 这个 URL 包含 redirect_uri，即登录成功后返回的地址。
     <br>

3. 浏览器被重定向到身份提供商的登录页面:

   - 例如，Azure AD 登录 URL：

   ```bash
   https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/authorize
   ?client_id=YOUR_CLIENT_ID
   &response_type=code
   &redirect_uri=https://yourapp.azurewebsites.net/.auth/login/aad/callback
   &scope=SCOPE
   ```

4. 用户输入凭据，身份提供商认证成功后，重定向回 Easy Auth 处理的回调 URL

   ```bash
   https://yourapp.azurewebsites.net/.auth/login/aad/callback?code=AUTHORIZATION_CODE
   ```

   Authorization Code 由身份提供商颁发。

5. 浏览器按照 302 重定向请求的 URL，访问前端服务
   这一步仍然是在 Easy Auth Middleware 里处理。

6. Easy Auth Middleware 处理请求，并使用 Authorization Code 向身份提供商获取 access Token / id token / refresh token / expires in
   Easy Auth 负责存储 Token，不会直接暴露给前端

   - 例如，Azure AD 发送的请求:

   ```bash
   POST https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token
    Content-Type: application/x-www-form-urlencoded

    client_id=YOUR_CLIENT_ID
    &grant_type=authorization_code
    &code=AUTHORIZATION_CODE
    &redirect_uri=https://yourapp.azurewebsites.net/.auth/login/aad/callback
    &client_secret=YOUR_CLIENT_SECRET
   ```

   - 返回的 Token 信息:

   ```json
   {
     "access_token": "ACCESS_TOKEN",
     "id_token": "ID_TOKEN",
     "refresh_token": "REFRESH_TOKEN",
     "expires_in": 3600
   }
   ```

7. Easy Auth Middleware 生成 Session Token，并通过 Set-Cookie 头返回给浏览器
   Session Token (Cookie: AppServiceAuthSession)用于后续请求的身份验证：

8. 浏览器在后续 Http 请求中包含 Session Token（Cookie），并访问前端服务。

9. Easy Auth Middleware 拦截请求，检测到 Session Token, 并确认是否过期。

- 如果 Session Token 过期，返回 302 重定向到身份提供商的登录页面。(跳转到步骤 2)

11. Easy Auth Middleware 根据 Session Token 中的信息，从存储中获取 access Token / id token / refresh token / expires in， 并将其添加到 Http Header 中。

12. 前端服务从 Http Header 中获取 access token 并判断其是否有效
    access token 的有效期可以使用 jwt decode 后的 exp 字段判断，也可以从 http header 中获取 expires_in 字段判断。

13. 前端服务使用 access token 调用后端服务的 API。
    并将 access token 使用 Bearer 认证方式添加到 Http Header 中。

    ```bash
    Authorization: Bearer ACCESS_TOKEN
    ```

14. 后端服务的 Easy Auth Middleware 拦截请求，检测到 header 中的 Bearer access token, 并确认是否有效。
    如果 access token 过期/无效/不存在，返回 401 错误。
    如果 access token 有效，Easy Auth Middleware 从 Http Header 中获取 access token 并将其添加到 Http Header 中。(与步骤 10 类似)

15. 后端服务的逻辑获取 header 中的 Bearer access token。

16. 后端服务使用 access token 调用 IdP 服务的 API 并获取用户信息。

17. 后端服务如果需要调用另一个后端服务的 API，也需要将 access token 使用 Bearer 认证方式添加到 Http Header 中。
    ```bash
    Authorization: Bearer ACCESS_TOKEN
    ```

### Access Token 过期后刷新的逻辑

![sequence]({{ "/assets/images/2025-03-20/refresh.jpg" | absolute url }})

1. 浏览器在 Http 请求中包含 Session Token（Cookie），并访问前端服务。

2. Easy Auth Middleware 拦截请求，检测到 Session Token, 并确认是否过期。

   - 如果 Session Token 过期，返回 302 重定向到身份提供商的登录页面。

3. Easy Auth Middleware 根据 Session Token 有效, 并根据 Session Token，从存储中获取 access Token / id token / refresh token / expires in， 并将其添加到 Http Header 中。

4. 前端服务从 Http Header 中获取 access token 并判断其是否有效(此时发现已过期)

5. 前端服务发现 access token 过期，返回 207 状态码

6. 浏览器收到 207 后，调用/.auth/refresh 接口刷新 access token

7. refresh 请求被 Easy Auth Middleware 处理，从 IdP 拿到刷新后 access token

8. Easy Auth Middleware 返回 http 200 状态码

9. 浏览器收到 200 后，重新进行步骤 1 的 http 请求

10. 同步骤 2

11. 同步骤 3

12. 前端服务从 Http Header 中获取 access token 并判断其是否有效(此时发现未过期)

### 我们遇到的刷新 token 失败的问题

![refresh-fail]({{ "/assets/images/2025-03-20/refresh-failed.jpg" | absolute url }})

说明: 我们遇到的现象是，只有前端服务对应的 App Service Plan 启用了 Scale out 到多个 instance，才会出现刷新 token 失败的问题。

步骤 1 ～ 11 与上面的流程图一致。
步骤 12 时，前端服务从 Http Header 中获取 access token 并判断其是否有效
此时发现的 token 与步骤 4 拿到的 token 内容一模一样，仍然是无效的， 并没有拿到刷新后的 token。

这看起来非常奇怪

按照我们的多次测试，以及对官方文档的学习，加上和 Azure 技术支持的沟通，我们猜测的情况如下。

![refresh-fail]({{ "/assets/images/2025-03-20/refresh-failed-1.jpg" | absolute url }})

![refresh-fail]({{ "/assets/images/2025-03-20/refresh-failed-2.jpg" | absolute url }})

简单的说，也就是因为多个 instance 下的 access token 和 token 的 refresh 操作不同步，导致刷新 token 失败。

### 解决方案

其中一种解决方案是启用 ARR (Application Request Routing)，将所有具有相同 Session Token Cookie 的请求都路由到同一个 instance 下。

![ARR]({{ "/assets/images/2025-03-20/arr.jpg" | absolute url }})

这样可以保证多个 instance 下的 access token 和 token 的 refresh 操作不会出现不同步的问题。

![refresh-fail]({{ "/assets/images/2025-03-20/arr-fix-issue.jpg" | absolute url }})

### 一些思考

![Architechture]({{ "/assets/images/2025-03-20/architecture.png" | absolute url }})

如果多个 App Service Instance 会导致 token refresh 和 token 管理不同步的话，和这个图的描述是否有矛盾呢？
从这个图上看 token store 是一个共享的资源，为什么会导致不同步呢？

### 关于通过将 key store 从 file 修改为 blob 来规避多个 instance 下 accesskey 不同步问题的文档

我们找到了如下文档，里面提到如果不启用 ARR 的话，也可以通过将 key store 从 file 修改为 blob 来规避多个 instance 下 accesskey 不同步问题。

<https://stackoverflow.com/questions/69385054/azure-app-service-authentication-token-not-refreshing-after-calling-auth-refre>

<https://johnnyreilly.com/easy-auth-tokens-survive-releases-on-linux-azure-app-service>

目前我们正在尝试这个方案

按我们的假设, 可能可以这么理解

使用 file storage 作为 key store 的话

![refresh-fail]({{ "/assets/images/2025-03-20/file-storage-token-store.jpg" | absolute url }})

使用 blob storage 作为 key store 的话

![refresh-fail]({{ "/assets/images/2025-03-20/blob-storage-token-store.jpg" | absolute url }})
