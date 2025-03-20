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

### 关于通过将 key store 从 file 修改为 blob 来规避多个 instance 下 accesskey 不同步问题的文档

<https://stackoverflow.com/questions/69385054/azure-app-service-authentication-token-not-refreshing-after-calling-auth-refre>
<https://johnnyreilly.com/easy-auth-tokens-survive-releases-on-linux-azure-app-service>
