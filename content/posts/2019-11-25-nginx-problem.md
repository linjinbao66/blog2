---
title: nginx代理出来的页面使用IE无法加载png问题
tags:
  - http
  - nginx
date: 2019-11-25
---

## nginx代理出来的页面使用IE无法加载png问题

问题描述： 使用nginx对站点进行反代，其他都正常，但是有个png图片无法加载。排查了情况如下： 1. 其他浏览器正常 2. IE在内网正常，访问原始地址 3. IE在外网异常，访问nginx代理后地址

问题分析： 检查F12，发现图片状态码200，已经加载出来，但是没有渲染出来，推测是由于安全策略问题；比对内网header和外网访问header，发现区别在外网多了个`X-Content-Type-Options: nosniff`，估计问题在这。

问题解决： 在nginx代理中，去掉`add_header X-Content-Type-Options nosniff;`配置项 问题解决

## 总结

`X-Content-Type-Options nosniff`是什么？

如果服务器发送响应头 "X-Content-Type-Options: nosniff"，则 script 和 styleSheet 元素会拒绝包含错误的 MIME 类型的响应。这是一种安全功能，有助于防止基于 MIME 类型混淆的攻击。

如果通过 styleSheet 参考检索到的响应中接收到 "nosniff" 指令，则 Windows Internet Explorer 不会加载“stylesheet”文件，除非 MIME 类型匹配 "text/css"。

如果通过 script 参考检索到的响应中接收到 "nosniff" 指令，则 Internet Explorer 不会加载“script”文件，除非 MIME 类型匹配以下值之一：

```text
"application/ecmascript"

"application/javascript"

"application/x-javascript"

"text/ecmascript"

"text/javascript"

"text/jscript"

"text/x-javascript"

"text/vbs"

"text/vbscript"
```

解决方案： 1. 使用`image/png`明确页面内容，同时启用`X-Content-Type-Options: nosniff`，禁用浏览器类型猜测行为 2. 禁用`X-Content-Type-Options: nosniff`，允许浏览器猜测行为
