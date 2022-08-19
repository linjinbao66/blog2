---
title: nginx拦截空reffer
date: 2020-11-10
---

## 背景解释

web系统出现垂直越权的现象，具体表现为，低权限账户登录后，直接请求高权限的接口可以获取数据，原因在于后台并没有实现对接口的权限校验，只是在菜单上简单隐藏。

## 解决方案

方案一：引入完善的权限框架，例如shiro

方案二：nginx控制reffer，禁止直接访问

配置实例：

```code
location /login.html {
    root        D:\code2\fdmp-pages;
    autoindex on;
    index  index.html;
    }

    location / {
    valid_referers server_names ~.;       ##校验所有请求的reffer 
    if ($invalid_referer) {     ##非法则返回403
    return 403;
    }
    root        D:\code2\fdmp-pages;
    autoindex on;
    index  index.html;
}
```
